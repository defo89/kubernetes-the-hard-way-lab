# Kubernetes Configuration Files for Authentication

## Client Authentication Configs

### kubelet Kubernetes Configuration File

```
{
for i in k8s-worker1 k8s-worker2 k8s-worker3; do
  kubectl config set-cluster kubernetes \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://10.98.95.18:6443 \
    --kubeconfig=${i}.kubeconfig

  kubectl config set-credentials system:node:${i} \
    --client-certificate=${i}.pem \
    --client-key=${i}-key.pem \
    --embed-certs=true \
    --kubeconfig=${i}.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes \
    --user=system:node:${i} \
    --kubeconfig=${i}.kubeconfig

  kubectl config use-context default --kubeconfig=${i}.kubeconfig
done
}
```

Result

```
k8s-worker1.kubeconfig
k8s-worker2.kubeconfig
k8s-worker3.kubeconfig
```

### kube-proxy Kubernetes Configuration File

```
{
kubectl config set-cluster kubernetes-the-hard-way \
--certificate-authority=ca.pem \
--embed-certs=true \
--server=https://10.98.95.18:6443 \
--kubeconfig=kube-proxy.kubeconfig

kubectl config set-credentials system:kube-proxy \
--client-certificate=kube-proxy.pem \
--client-key=kube-proxy-key.pem \
--embed-certs=true \
--kubeconfig=kube-proxy.kubeconfig

kubectl config set-context default \
--cluster=kubernetes-the-hard-way \
--user=system:kube-proxy \
--kubeconfig=kube-proxy.kubeconfig

kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
}
```

Result

```
kube-proxy.kubeconfig
```

### kube-controller-manager Kubernetes Configuration File

```
{
kubectl config set-cluster kubernetes-the-hard-way \
--certificate-authority=ca.pem \
--embed-certs=true \
--server=https://127.0.0.1:6443 \
--kubeconfig=kube-controller-manager.kubeconfig

kubectl config set-credentials system:kube-controller-manager \
--client-certificate=kube-controller-manager.pem \
--client-key=kube-controller-manager-key.pem \
--embed-certs=true \
--kubeconfig=kube-controller-manager.kubeconfig

kubectl config set-context default \
--cluster=kubernetes-the-hard-way \
--user=system:kube-controller-manager \
--kubeconfig=kube-controller-manager.kubeconfig

kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
}
```

Result

```
kube-controller-manager.kubeconfig
```

### kube-scheduler Kubernetes Configuration File

```
{
kubectl config set-cluster kubernetes-the-hard-way \
--certificate-authority=ca.pem \
--embed-certs=true \
--server=https://127.0.0.1:6443 \
--kubeconfig=kube-scheduler.kubeconfig

kubectl config set-credentials system:kube-scheduler \
--client-certificate=kube-scheduler.pem \
--client-key=kube-scheduler-key.pem \
--embed-certs=true \
--kubeconfig=kube-scheduler.kubeconfig

kubectl config set-context default \
--cluster=kubernetes-the-hard-way \
--user=system:kube-scheduler \
--kubeconfig=kube-scheduler.kubeconfig

kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
}
```

Result

```
kube-scheduler.kubeconfig
```

### admin Kubernetes Configuration File

```
{
kubectl config set-cluster kubernetes-the-hard-way \
--certificate-authority=ca.pem \
--embed-certs=true \
--server=https://127.0.0.1:6443 \
--kubeconfig=admin.kubeconfig

kubectl config set-credentials admin \
--client-certificate=admin.pem \
--client-key=admin-key.pem \
--embed-certs=true \
--kubeconfig=admin.kubeconfig

kubectl config set-context default \
--cluster=kubernetes-the-hard-way \
--user=admin \
--kubeconfig=admin.kubeconfig

kubectl config use-context default --kubeconfig=admin.kubeconfig
}
```

Result

```
admin.kubeconfig
```

## Distribute the Kubernetes Configuration Files

```
for i in k8s-controller1 k8s-controller2 k8s-controller3; do
scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig defo@${i}:~/
done

for i in k8s-worker1 k8s-worker2 k8s-worker3; do
scp ${i}.kubeconfig kube-proxy.kubeconfig defo@${i}:~/
done
```



