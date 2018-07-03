# Configure Kubernetes Networking

## Configure Kubernetes Networking - Calico

Run from admin machine.

Project Calico will be used for Kubernetes networking. 

### Configure the roles and bindings that Calico requires

```
kubectl apply -f \
https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
```

Output

```
clusterrole "calico-node" created
clusterrolebinding "calico-node" created
```

### Install Calico

```
kubectl apply -f \
https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
```

Output

```
configmap "calico-config" created
service "calico-typha" created
deployment "calico-typha" created
daemonset "calico-node" created
customresourcedefinition "felixconfigurations.crd.projectcalico.org" created
customresourcedefinition "bgppeers.crd.projectcalico.org" created
customresourcedefinition "bgpconfigurations.crd.projectcalico.org" created
customresourcedefinition "ippools.crd.projectcalico.org" created
customresourcedefinition "hostendpoints.crd.projectcalico.org" created
customresourcedefinition "clusterinformations.crd.projectcalico.org" created
customresourcedefinition "globalnetworkpolicies.crd.projectcalico.org" created
customresourcedefinition "globalnetworksets.crd.projectcalico.org" created
customresourcedefinition "networkpolicies.crd.projectcalico.org" created
serviceaccount "calico-node" created
```

### Verification

Install calicoctl

```
brew install calicoctl
```

```
> DATASTORE_TYPE=kubernetes KUBECONFIG=~/.kube/config calicoctl get nodes
NAME
k8s-worker1
k8s-worker2
k8s-worker3
```

```
> DATASTORE_TYPE=kubernetes KUBECONFIG=~/.kube/config calicoctl get nodes -o wide
NAME          ASN         IPV4             IPV6
k8s-worker1   (unknown)   10.98.95.21/24
k8s-worker2   (unknown)   10.98.95.22/24
k8s-worker3   (unknown)   10.98.95.23/24
```

```
> DATASTORE_TYPE=kubernetes KUBECONFIG=~/.kube/config calicoctl get workloadendpoints --all
NAMESPACE     WORKLOAD                   NODE          NETWORKS         INTERFACE
default       busybox-68654f944b-qsn9m   k8s-worker2   10.98.101.6/32   cali6d834f43414
kube-system   coredns-75b467dc56-8n44m   k8s-worker2   10.98.101.7/32   calid4cc8c49012
kube-system   coredns-75b467dc56-f4f78   k8s-worker2
```

## Configure Kubernetes Networking - Flannel

### Install Flannel

```
kubectl apply -f \
https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Output

```
clusterrole "flannel" created
clusterrolebinding "flannel" created
serviceaccount "flannel" created
configmap "kube-flannel-cfg" created
daemonset "kube-flannel-ds" created
```

## Configure Kubernetes Networking - Weave Net

### Install Weave Net

```
kubectl apply -f \
"https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

Output

```
serviceaccount "weave-net" configured
clusterrole "weave-net" configured
clusterrolebinding "weave-net" configured
role "weave-net" created
rolebinding "weave-net" created
daemonset "weave-net" created
```

## Troubleshooting

Make sure that networking solutions don't conflict in folders

```
/etc/cni/net.d/
/opt/cni/bin
```

In case of changing the solution on Worker node

```
sudo rm -f /etc/cni/net.d/*
sudo shutdown -r now
```