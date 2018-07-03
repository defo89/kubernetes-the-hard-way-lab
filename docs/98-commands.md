# Verification and Troubleshooting

## Cluster-info

```
kubectl cluster-info
```

```
> kubectl cluster-info
Kubernetes master is running at https://10.98.95.18:6443
CoreDNS is running at https://10.98.95.18:6443/api/v1/namespaces/kube-system/services/kube-dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

## Get

```
kubectl get node
```

```
> kubectl get node
NAME          STATUS     ROLES     AGE       VERSION
k8s-worker1   NotReady   <none>    1h        v1.10.2
k8s-worker2   NotReady   <none>    1h        v1.10.2
k8s-worker3   NotReady   <none>    1h        v1.10.2
```

```
kubectl get nodes -o wide
```

```
> kubectl get nodes -o wide
NAME          STATUS     ROLES     AGE       VERSION   EXTERNAL-IP   OS-IMAGE           KERNEL-VERSION      CONTAINER-RUNTIME
k8s-worker1   NotReady   <none>    1h        v1.10.2   <none>        Ubuntu 18.04 LTS   4.15.0-23-generic   containerd://1.1.0
k8s-worker2   NotReady   <none>    1h        v1.10.2   <none>        Ubuntu 18.04 LTS   4.15.0-23-generic   containerd://1.1.0
k8s-worker3   NotReady   <none>    1h        v1.10.2   <none>        Ubuntu 18.04 LTS   4.15.0-23-generic   containerd://1.1.0
```

```
kubectl get namespace
```

```
> kubectl get namespace
NAME          STATUS    AGE
default       Active    17h
kube-public   Active    17h
kube-system   Active    17h
```

```
kubectl get pod --namespace kube-system
```

```
> kubectl get pod --namespace kube-system
NAME                       READY     STATUS    RESTARTS   AGE
calico-node-cxtv2          0/2       Blocked   0          55m
calico-node-qwsz2          0/2       Blocked   0          55m
calico-node-vx9h9          0/2       Blocked   0          55m
coredns-75b467dc56-cxj7b   0/1       Pending   0          12m
coredns-75b467dc56-rg5t9   0/1       Pending   0          12m

> kubectl get pod --namespace kube-public
No resources found.

> kubectl get pod --namespace default
No resources found.
```

```
kubectl get services --all-namespaces
```

```
> kubectl get services --all-namespaces
NAMESPACE     NAME           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)         AGE
default       kubernetes     ClusterIP   10.98.96.1     <none>        443/TCP         1d
kube-system   calico-typha   ClusterIP   10.98.96.248   <none>        5473/TCP        19h
kube-system   kube-dns       ClusterIP   10.98.96.10    <none>        53/UDP,53/TCP   18h
```

```
kubectl get componentstatuses
```

```
> kubectl get componentstatuses
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok
etcd-0               Healthy   {"health":"true"}
etcd-1               Healthy   {"health":"true"}
etcd-2               Healthy   {"health":"true"}
controller-manager   Healthy   ok
```

```
kubectl get configmap -n kube-system
```

```
> kubectl get configmap -n kube-system
NAME                                 DATA      AGE
calico-config                        2         20h
coredns                              1         19h
extension-apiserver-authentication   1         1d
```

```
kubectl -n kube-system get configmap coredns -o yaml
```

```
> kubectl -n kube-system get configmap coredns -o yaml
apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health
        kubernetes cluster.local 10.98.100.0/22 10.98.96.0/24 {
          pods insecure
          upstream
          fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        proxy . /etc/resolv.conf
        cache 30
    }
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"Corefile":".:53 {\n    errors\n    health\n    kubernetes cluster.local 10.98.100.0/22 10.98.96.0/24 {\n      pods insecure\n      upstream\n      fallthrough in-addr.arpa ip6.arpa\n    }\n    prometheus :9153\n    proxy . /etc/resolv.conf\n    cache 30\n}\n"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"coredns","namespace":"kube-system"}}
  creationTimestamp: 2018-07-01T14:23:25Z
  name: coredns
  namespace: kube-system
  resourceVersion: "64205"
  selfLink: /api/v1/namespaces/kube-system/configmaps/coredns
  uid: 50e2363c-7d3a-11e8-98c3-000c29e99a17
```

```
kubectl get all --all-namespaces
```

```
> kubectl get all --all-namespaces
NAMESPACE     NAME             AGE
kube-system   ds/calico-node   50m

NAMESPACE     NAME                  AGE
kube-system   deploy/calico-typha   50m
kube-system   deploy/coredns        7m

NAMESPACE     NAME                         AGE
kube-system   rs/calico-typha-5d6c99878c   50m
kube-system   rs/coredns-75b467dc56        7m

NAMESPACE     NAME                          READY     STATUS    RESTARTS   AGE
kube-system   po/calico-node-cxtv2          0/2       Blocked   0          50m
kube-system   po/calico-node-qwsz2          0/2       Blocked   0          50m
kube-system   po/calico-node-vx9h9          0/2       Blocked   0          50m
kube-system   po/coredns-75b467dc56-cxj7b   0/1       Pending   0          7m
kube-system   po/coredns-75b467dc56-rg5t9   0/1       Pending   0          7m

NAMESPACE     NAME               TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)         AGE
default       svc/kubernetes     ClusterIP   10.98.96.1     <none>        443/TCP         17h
kube-system   svc/calico-typha   ClusterIP   10.98.96.248   <none>        5473/TCP        50m
kube-system   svc/kube-dns       ClusterIP   10.98.96.10    <none>        53/UDP,53/TCP   7m
```

## Describe

```
kubectl describe service
```

```
> kubectl describe service kubernetes
Name:              kubernetes
Namespace:         default
Labels:            component=apiserver
                   provider=kubernetes
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP:                10.98.96.1
Port:              https  443/TCP
TargetPort:        6443/TCP
Endpoints:         10.98.95.11:6443,10.98.95.12:6443,10.98.95.13:6443
Session Affinity:  ClientIP
Events:            <none>
```

```
kubectl describe nodes
```

```
kubectl describe nodes
Name:               k8s-worker1
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/hostname=k8s-worker1
Annotations:        node.alpha.kubernetes.io/ttl=0
                    volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:             <none>
CreationTimestamp:  Sun, 01 Jul 2018 16:08:03 +0300
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  OutOfDisk        False   Sun, 01 Jul 2018 17:38:43 +0300   Sun, 01 Jul 2018 16:33:02 +0300   KubeletHasSufficientDisk     kubelet has sufficient disk space available
  MemoryPressure   False   Sun, 01 Jul 2018 17:38:43 +0300   Sun, 01 Jul 2018 16:33:02 +0300   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Sun, 01 Jul 2018 17:38:43 +0300   Sun, 01 Jul 2018 16:33:02 +0300   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Sun, 01 Jul 2018 17:38:43 +0300   Sun, 01 Jul 2018 16:08:03 +0300   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            False   Sun, 01 Jul 2018 17:38:43 +0300   Sun, 01 Jul 2018 16:33:02 +0300   KubeletNotReady              runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni config load failed: no network config found in /etc/cni/net.d: cni plugin not initialized: failed to load cni config
Addresses:
  InternalIP:  10.98.95.21
  Hostname:    k8s-worker1
Capacity:
 cpu:                1
 ephemeral-storage:  16445308Ki
 hugepages-2Mi:      0
 memory:             1009644Ki
 pods:               110
Allocatable:
 cpu:                1
 ephemeral-storage:  15155995828
 hugepages-2Mi:      0
 memory:             907244Ki
 pods:               110
System Info:
 Machine ID:                 1033dd45c760425cab83fd68b3dcbc15
 System UUID:                85F94D56-40F9-3EC1-2BE8-1854A89CB6AB
 Boot ID:                    15e76b06-1711-4b97-a8c8-cbe5249cc4f5
 Kernel Version:             4.15.0-23-generic
 OS Image:                   Ubuntu 18.04 LTS
 Operating System:           linux
 Architecture:               amd64
 Container Runtime Version:  containerd://1.1.0
 Kubelet Version:            v1.10.2
 Kube-Proxy Version:         v1.10.2
PodCIDR:                     10.98.100.0/24
ExternalID:                  k8s-worker1
Non-terminated Pods:         (1 in total)
  Namespace                  Name                 CPU Requests  CPU Limits  Memory Requests  Memory Limits
  ---------                  ----                 ------------  ----------  ---------------  -------------
  kube-system                calico-node-vx9h9    250m (25%)    0 (0%)      0 (0%)           0 (0%)
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  CPU Requests  CPU Limits  Memory Requests  Memory Limits
  ------------  ----------  ---------------  -------------
  250m (25%)    0 (0%)      0 (0%)           0 (0%)
Events:         <none>


Name:               k8s-worker2
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/hostname=k8s-worker2
Annotations:        node.alpha.kubernetes.io/ttl=0
                    volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:             <none>
CreationTimestamp:  Sun, 01 Jul 2018 16:08:00 +0300
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  OutOfDisk        False   Sun, 01 Jul 2018 17:38:49 +0300   Sun, 01 Jul 2018 16:33:08 +0300   KubeletHasSufficientDisk     kubelet has sufficient disk space available
  MemoryPressure   False   Sun, 01 Jul 2018 17:38:49 +0300   Sun, 01 Jul 2018 16:33:08 +0300   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Sun, 01 Jul 2018 17:38:49 +0300   Sun, 01 Jul 2018 16:33:08 +0300   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Sun, 01 Jul 2018 17:38:49 +0300   Sun, 01 Jul 2018 16:07:59 +0300   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            False   Sun, 01 Jul 2018 17:38:49 +0300   Sun, 01 Jul 2018 16:33:08 +0300   KubeletNotReady              runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni config load failed: no network config found in /etc/cni/net.d: cni plugin not initialized: failed to load cni config
Addresses:
  InternalIP:  10.98.95.22
  Hostname:    k8s-worker2
Capacity:
 cpu:                1
 ephemeral-storage:  16445308Ki
 hugepages-2Mi:      0
 memory:             1009644Ki
 pods:               110
Allocatable:
 cpu:                1
 ephemeral-storage:  15155995828
 hugepages-2Mi:      0
 memory:             907244Ki
 pods:               110
System Info:
 Machine ID:                 ee585d1be06c44c581ee4dc66865f485
 System UUID:                79E54D56-2F3A-A4BF-12E1-68F398C88C8A
 Boot ID:                    5f4d8747-3ca2-44cb-a293-27f6dc3cdef1
 Kernel Version:             4.15.0-23-generic
 OS Image:                   Ubuntu 18.04 LTS
 Operating System:           linux
 Architecture:               amd64
 Container Runtime Version:  containerd://1.1.0
 Kubelet Version:            v1.10.2
 Kube-Proxy Version:         v1.10.2
PodCIDR:                     10.98.101.0/24
ExternalID:                  k8s-worker2
Non-terminated Pods:         (1 in total)
  Namespace                  Name                 CPU Requests  CPU Limits  Memory Requests  Memory Limits
  ---------                  ----                 ------------  ----------  ---------------  -------------
  kube-system                calico-node-cxtv2    250m (25%)    0 (0%)      0 (0%)           0 (0%)
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  CPU Requests  CPU Limits  Memory Requests  Memory Limits
  ------------  ----------  ---------------  -------------
  250m (25%)    0 (0%)      0 (0%)           0 (0%)
Events:         <none>


Name:               k8s-worker3
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/hostname=k8s-worker3
Annotations:        node.alpha.kubernetes.io/ttl=0
                    volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:             <none>
CreationTimestamp:  Sun, 01 Jul 2018 16:08:00 +0300
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  OutOfDisk        False   Sun, 01 Jul 2018 17:38:49 +0300   Sun, 01 Jul 2018 16:33:00 +0300   KubeletHasSufficientDisk     kubelet has sufficient disk space available
  MemoryPressure   False   Sun, 01 Jul 2018 17:38:49 +0300   Sun, 01 Jul 2018 16:33:00 +0300   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Sun, 01 Jul 2018 17:38:49 +0300   Sun, 01 Jul 2018 16:33:00 +0300   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Sun, 01 Jul 2018 17:38:49 +0300   Sun, 01 Jul 2018 16:08:00 +0300   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            False   Sun, 01 Jul 2018 17:38:49 +0300   Sun, 01 Jul 2018 16:33:00 +0300   KubeletNotReady              runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni config load failed: no network config found in /etc/cni/net.d: cni plugin not initialized: failed to load cni config
Addresses:
  InternalIP:  10.98.95.23
  Hostname:    k8s-worker3
Capacity:
 cpu:                1
 ephemeral-storage:  16445308Ki
 hugepages-2Mi:      0
 memory:             1009644Ki
 pods:               110
Allocatable:
 cpu:                1
 ephemeral-storage:  15155995828
 hugepages-2Mi:      0
 memory:             907244Ki
 pods:               110
System Info:
 Machine ID:                 1bd0abb1ffa344b68bf3329852af19de
 System UUID:                55A34D56-8117-588D-F66F-6A64EFE99B9B
 Boot ID:                    229d9333-e0ab-4c12-8bd4-5f440ac0bb7a
 Kernel Version:             4.15.0-23-generic
 OS Image:                   Ubuntu 18.04 LTS
 Operating System:           linux
 Architecture:               amd64
 Container Runtime Version:  containerd://1.1.0
 Kubelet Version:            v1.10.2
 Kube-Proxy Version:         v1.10.2
PodCIDR:                     10.98.102.0/24
ExternalID:                  k8s-worker3
Non-terminated Pods:         (1 in total)
  Namespace                  Name                 CPU Requests  CPU Limits  Memory Requests  Memory Limits
  ---------                  ----                 ------------  ----------  ---------------  -------------
  kube-system                calico-node-qwsz2    250m (25%)    0 (0%)      0 (0%)           0 (0%)
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  CPU Requests  CPU Limits  Memory Requests  Memory Limits
  ------------  ----------  ---------------  -------------
  250m (25%)    0 (0%)      0 (0%)           0 (0%)
Events:         <none>
```

## Exec

```
kubectl exec
```

```
> kubectl get pods -l run=busybox

NAME                       READY     STATUS    RESTARTS   AGE
busybox-68654f944b-9hl4c   1/1       Running   0          21s

> POD_NAME=$(kubectl get pods -l run=busybox -o jsonpath="{.items[0].metadata.name}")
> echo $POD_NAME
busybox-68654f944b-9hl4c

> kubectl exec -ti $POD_NAME -- ifconfig eth0
eth0      Link encap:Ethernet  HWaddr B2:D5:E1:38:38:2E
          inet addr:10.98.102.2  Bcast:0.0.0.0  Mask:255.255.255.255
          inet6 addr: fe80::b0d5:e1ff:fe38:382e/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:55 errors:0 dropped:0 overruns:0 frame:0
          TX packets:57 errors:0 dropped:1 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:5470 (5.3 KiB)  TX bytes:4742 (4.6 KiB)
```

## Logs

```
kubectl logs coredns-75b467dc56-cxj7b -n kube-system
```

```
> kubectl logs coredns-75b467dc56-cxj7b -n kube-system
E0702 08:44:32.982830       1 reflector.go:205] github.com/coredns/coredns/plugin/kubernetes/controller.go:320: Failed to list *v1.Namespace: Get https://10.98.96.1:443/api/v1/namespaces?limit=500&resourceVersion=0: dial tcp 10.98.96.1:443: i/o timeout
E0702 08:44:32.991530       1 reflector.go:205] github.com/coredns/coredns/plugin/kubernetes/controller.go:313: Failed to list *v1.Service: Get https://10.98.96.1:443/api/v1/services?limit=500&resourceVersion=0: dial tcp 10.98.96.1:443: i/o timeout
E0702 08:44:32.995851       1 reflector.go:205] github.com/coredns/coredns/plugin/kubernetes/controller.go:315: Failed to list *v1.Endpoints: Get https://10.98.96.1:443/api/v1/endpoints?limit=500&resourceVersion=0: dial tcp 10.98.96.1:443: i/o timeout
E0702 08:45:03.984418       1 reflector.go:205] github.com/coredns/coredns/plugin/kubernetes/controller.go:320: Failed to list *v1.Namespace: Get https://10.98.96.1:443/api/v1/namespaces?limit=500&resourceVersion=0: dial tcp 10.98.96.1:443: i/o timeout
E0702 08:45:03.994920       1 reflector.go:205] github.com/coredns/coredns/plugin/kubernetes/controller.go:313: Failed to list *v1.Service: Get https://10.98.96.1:443/api/v1/services?limit=500&resourceVersion=0: dial tcp 10.98.96.1:443: i/o timeout
E0702 08:45:03.998744       1 reflector.go:205] github.com/coredns/coredns/plugin/kubernetes/controller.go:315: Failed to list *v1.Endpoints: Get https://10.98.96.1:443/api/v1/endpoints?limit=500&resourceVersion=0: dial tcp 10.98.96.1:443: i/o timeout
E0702 08:45:34.985422       1 reflector.go:205] github.com/coredns/coredns/plugin/kubernetes/controller.go:320: Failed to list *v1.Namespace: Get https://10.98.96.1:443/api/v1/namespaces?limit=500&resourceVersion=0: dial tcp 10.98.96.1:443: i/o timeout
E0702 08:45:34.997418       1 reflector.go:205] github.com/coredns/coredns/plugin/kubernetes/controller.go:313: Failed to list *v1.Service: Get https://10.98.96.1:443/api/v1/services?limit=500&resourceVersion=0: dial tcp 10.98.96.1:443: i/o timeout
E0702 08:45:35.000085       1 reflector.go:205] github.com/coredns/coredns/plugin/kubernetes/controller.go:315: Failed to list *v1.Endpoints: Get https://10.98.96.1:443/api/v1/endpoints?limit=500&resourceVersion=0: dial tcp 10.98.96.1:443: i/o timeout
2018/07/02 08:45:42 [INFO] SIGTERM: Shutting down servers then terminating
```
