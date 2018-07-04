# Testing and Playing with Kubernetes

## Sonoboy Diagnostics

Sonoboy is Heptio's diagnostic tool for running Kubernetes conformance tests.

```
https://scanner.heptio.com
```

## Helm

```
brew install kubernetes-helm
helm init
```

## Kubernetes Dashboard

https://github.com/kubernetes/dashboard

```
kubectl apply -f \
https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```

Accessing Web

```
kubectl proxy
```

```
http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
```

## Prometheus Operator

### Installation

```
helm repo add coreos https://s3-eu-west-1.amazonaws.com/coreos-charts/stable/
helm install coreos/prometheus-operator --set rbacEnable=false --name prometheus-operator --namespace monitoring
helm install coreos/kube-prometheus --name kube-prometheus --set global.rbacEnable=true --namespace monitoring
```

### Verification

```
kubectl get pods  -n monitoring
```

## MetalLB for Load Balancing

### Installation

```
kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.6.2/manifests/metallb.yaml
```

Output

```
namespace "metallb-system" created
serviceaccount "controller" created
serviceaccount "speaker" created
clusterrole "metallb-system:controller" created
clusterrole "metallb-system:speaker" created
role "leader-election" created
role "config-watcher" created
clusterrolebinding "metallb-system:controller" created
clusterrolebinding "metallb-system:speaker" created
rolebinding "config-watcher" created
rolebinding "leader-election" created
daemonset "speaker" created
deployment "controller" created
```

### Configuration

```
kubectl apply -f metallb-config.yml
```

### Verification

```
kubectl get pods  -n metallb-system
```

Output

```
> kubectl get pods  -n metallb-system
NAME                          READY     STATUS    RESTARTS   AGE
controller-58b7d685df-6d2q9   1/1       Running   0          3m
speaker-6dr7k                 1/1       Running   0          3m
speaker-jcgpt                 1/1       Running   0          3m
speaker-tg6lp                 1/1       Running   0          3m
```