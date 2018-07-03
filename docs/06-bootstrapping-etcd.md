# Bootstrapping the etcd Cluster

## Bootstrapping an etcd Cluster Member

### Download and copy the etcd Binaries

```
wget -q --show-progress --https-only --timestamping \
  "https://github.com/coreos/etcd/releases/download/v3.3.8/etcd-v3.3.8-linux-amd64.tar.gz"

for i in k8s-controller1 k8s-controller2 k8s-controller3; do
scp etcd-v3.3.8-linux-amd64.tar.gz defo@${i}:~/
done
```

### Install the etcd Binaries

Commands will be run on each controller instance: k8s-controller1, k8s-controller2, k8s-controller3.

```
{
  tar -xvf etcd-v3.3.8-linux-amd64.tar.gz
  sudo mv etcd-v3.3.8-linux-amd64/etcd* /usr/local/bin/
}
```

### Configure the etcd Server

```
{
  sudo mkdir -p /etc/etcd /var/lib/etcd
  sudo cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
}
```

Configure variables

```
ETCD_NAME=`hostname -s`
INTERNAL_IP=`nslookup $(hostname -s) | awk '/^Address: / { print $2 }'`
```

Create the etcd.service systemd unit file

```
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/kubernetes.pem \\
  --key-file=/etc/etcd/kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/kubernetes.pem \\
  --peer-key-file=/etc/etcd/kubernetes-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster k8s-controller1=https://10.98.95.11:2380,k8s-controller2=https://10.98.95.12:2380,k8s-controller3=https://10.98.95.13:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the etcd Server

Run on each controller instance: k8s-controller1, k8s-controller2, k8s-controller3

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable etcd
  sudo systemctl start etcd
}
```

Output

```
Created symlink /etc/systemd/system/multi-user.target.wants/etcd.service → /etc/systemd/system/etcd.service.
```

### Verify etcd Server

List the etcd cluster members

```
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem
```

Output

```
940d3656ac8ca09, started, k8s-controller3, https://10.98.95.13:2380, https://10.98.95.13:2379
bb24a3911fa8ecae, started, k8s-controller2, https://10.98.95.12:2380, https://10.98.95.12:2379
dd2809b1471cbe4b, started, k8s-controller1, https://10.98.95.11:2380, https://10.98.95.11:2379
defo@k8s-controller1:~$ 
```