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