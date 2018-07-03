# Kubernetes Dashboard

https://github.com/kubernetes/dashboard

```
kubectl apply -f \
https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
````

````
kubectl proxy
````

Web
http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/