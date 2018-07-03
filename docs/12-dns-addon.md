# Deploying the DNS Cluster Add-on

## CoreDNS

CoreDNS is configured through a Corefile, and in Kubernetes, ConfigMap is used.

### Apply CoreDNS ConfigMap

```
kubectl apply -f coredns/coredns.yaml
```

Output

```
serviceaccount "coredns" created
clusterrole "system:coredns" created
clusterrolebinding "system:coredns" created
configmap "coredns" created
deployment "coredns" created
service "kube-dns" created
```

```
kubectl get pod --all-namespaces -l k8s-app=coredns -o wide
```

### Kube-DNS

Deploy the kube-dns cluster add-on

```
kubectl create -f kube-dns/kube-dns.yaml
```

Output

```
service "kube-dns" created
serviceaccount "kube-dns" created
configmap "kube-dns" created
deployment "kube-dns" created
```

List the pods created by the kube-dns deployment:

```
kubectl get pods -l k8s-app=kube-dns -n kube-system
```

Output

```
NAME                        READY     STATUS    RESTARTS   AGE
kube-dns-3097350089-gq015   3/3       Running   0          20s
```

## Verification

Create a busybox deployment

```
kubectl run busybox --image=busybox --command -- sleep 3600
```

List the pod created by the busybox deployment

```
kubectl get pods -l run=busybox
```

Output

```
NAME                       READY     STATUS    RESTARTS   AGE
busybox-68654f944b-9hl4c   1/1       Running   0          21s
```

Retrieve the full name of the busybox pod

```
POD_NAME=$(kubectl get pods -l run=busybox -o jsonpath="{.items[0].metadata.name}")

> echo $POD_NAME
busybox-68654f944b-9hl4c
```

Execute a DNS lookup for the kubernetes service inside the busybox pod:

```
kubectl exec -ti $POD_NAME -- nslookup kubernetes
```

Output

```
Server:    10.32.0.10
Address 1: 10.32.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes
Address 1: 10.32.0.1 kubernetes.default.svc.cluster.local
```