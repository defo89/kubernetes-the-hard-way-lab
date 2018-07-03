# Bootstrapping the Kubernetes Worker Nodes

## Download and copy official Install Worker Binaries

```
wget -q --show-progress --https-only --timestamping \
  https://github.com/kubernetes-incubator/cri-tools/releases/download/v1.0.0-beta.0/crictl-v1.0.0-beta.0-linux-amd64.tar.gz \
  https://storage.googleapis.com/kubernetes-the-hard-way/runsc \
  https://github.com/opencontainers/runc/releases/download/v1.0.0-rc5/runc.amd64 \
  https://github.com/containernetworking/plugins/releases/download/v0.6.0/cni-plugins-amd64-v0.6.0.tgz \
  https://github.com/containerd/containerd/releases/download/v1.1.0/containerd-1.1.0.linux-amd64.tar.gz \
  https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kubelet

for i in k8s-worker1 k8s-worker2 k8s-worker3; do
scp runc.amd64 kubectl kube-proxy kubelet runc runsc \
crictl-v1.0.0-beta.0-linux-amd64.tar.gz cni-plugins-amd64-v0.6.0.tgz \
containerd-1.1.0.linux-amd64.tar.gz defo@${i}:~/
done
```

## Prepare Worker Nodes

Commands will be run on each controller instance: k8s-worker1, k8s-worker2, k8s-worker3.

IPv4 packet forwarding should be enabled

```
/etc/sysctl.conf
net.ipv4.ip_forward=1

sudo sysctl -p /etc/sysctl.conf
```

Kubelet requires host to have no swap
Comment out swap in /etc/fstab file.

```
> sudo vim /etc/fstab
#/swap.img      none    swap    sw      0       0
> sudo shutdown -r now
```

Disable DNSStubListener

```
cat /etc/resolv.conf 
nameserver 127.0.0.53
search technoff.eu
```

```
> sudo vim /etc/systemd/resolved.conf
DNSStubListener=no
> sudo systemctl restart systemd-resolved
```


## Provisioning a Kubernetes Worker Node

```
{
  sudo apt-get update
  sudo apt-get -y install socat conntrack ipset
}
```

## Install Worker Binaries

Create the installation directories

```
sudo mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

Install the worker binaries

```
{
  chmod +x kubectl kube-proxy kubelet runc.amd64 runsc
  sudo mv runc.amd64 runc
  sudo mv kubectl kube-proxy kubelet runc runsc /usr/local/bin/
  sudo tar -xvf crictl-v1.0.0-beta.0-linux-amd64.tar.gz -C /usr/local/bin/
  sudo tar -xvf cni-plugins-amd64-v0.6.0.tgz -C /opt/cni/bin/
  sudo tar -xvf containerd-1.1.0.linux-amd64.tar.gz -C /
}
```

## Configure CNI Networking

We do not need to configure cni as we will setup Weave and it will do the necessary setup.

## Configure containerd (container runtime)

```
cat <<EOF | sudo tee /etc/systemd/system/containerd.service
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target

[Service]
ExecStartPre=/sbin/modprobe overlay
ExecStart=/bin/containerd
Restart=always
RestartSec=5
Delegate=yes
KillMode=process
OOMScoreAdjust=-999
LimitNOFILE=1048576
LimitNPROC=infinity
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
EOF
```

## Configure the Kubelet

```
{
  HOSTNAME=`hostname -s`
  sudo mv ${HOSTNAME}-key.pem ${HOSTNAME}.pem /var/lib/kubelet/
  sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
  sudo mv ca.pem /var/lib/kubernetes/
}
```

Create the kubelet-config.yaml configuration file

```
{
HOSTNAME=`hostname -s`

cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.98.96.10"
podCIDR: "10.98.100.0/22"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/${HOSTNAME}.pem"
tlsPrivateKeyFile: "/var/lib/kubelet/${HOSTNAME}-key.pem"
EOF
}
```

Create the kubelet.service systemd unit file

`--allow-privileged=true` - fixed `Waiting` state (specified privileged container, but is disallowed)
`--resolv-conf=/run/systemd/resolve/resolv.conf` - DNS Pod had `127.0.0.1` as a nameserver in `/etc/resolv.conf`, resolution was failing.

```
cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --container-runtime=remote \\
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --network-plugin=cni \\
  --register-node=true \\
  --allow-privileged=true \\
  --resolv-conf=/run/systemd/resolve/resolv.conf \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

## Configure the Kubernetes Proxy

```
sudo mv kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

Create the kube-proxy-config.yaml configuration file

```
cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "10.98.100.0/22"
EOF
```

Create the kube-proxy.service systemd unit file

```
cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

## Start the Worker Services

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable containerd kubelet kube-proxy
  sudo systemctl start containerd kubelet kube-proxy
}
```

Output

```
defo@k8s-worker1:~$ {
>   sudo systemctl daemon-reload
>   sudo systemctl enable containerd kubelet kube-proxy
>   sudo systemctl start containerd kubelet kube-proxy
> }
Created symlink /etc/systemd/system/multi-user.target.wants/containerd.service → /etc/systemd/system/containerd.service.
Created symlink /etc/systemd/system/multi-user.target.wants/kubelet.service → /etc/systemd/system/kubelet.service.
Created symlink /etc/systemd/system/multi-user.target.wants/kube-proxy.service → /etc/systemd/system/kube-proxy.service.
defo@k8s-worker1:~$ 
```

## Verification

```
kubectl get nodes --kubeconfig admin.kubeconfig
```

```
kubectl describe pod calico-node-cxtv2 --namespace kube-system
```

 ~/data/git/learning/kubernetes/k8s-lab  kubectl describe pod calico-node-cxtv2 --namespace kube-system
Name:           calico-node-cxtv2
Namespace:      kube-system
Node:           k8s-worker2/10.98.95.22
Start Time:     Sun, 01 Jul 2018 16:40:28 +0300
Labels:         controller-revision-hash=1808776410
                k8s-app=calico-node
                pod-template-generation=1
Annotations:    scheduler.alpha.kubernetes.io/critical-pod=
Status:         Pending
Reason:         Forbidden
Message:        pod with UID "50f6639a-7d34-11e8-be40-000c29e99a17" specified privileged container, but is disallowed




