# Configuring kubectl for Remote Access

Run from admin machine.

Generate a kubeconfig file for the kubectl command line utility based on the admin user credentials.

## Admin Kubernetes Configuration File

Generate a kubeconfig file suitable for authenticating as the admin user

```
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://10.98.95.18:6443

  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem

  kubectl config set-context kubernetes-the-hard-way \
    --cluster=kubernetes-the-hard-way \
    --user=admin

  kubectl config use-context kubernetes-the-hard-way
}
```

```
> kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: REDACTED
    server: https://10.98.95.18:6443
  name: kubernetes-the-hard-way
contexts:
- context:
    cluster: kubernetes-the-hard-way
    user: admin
  name: kubernetes-the-hard-way
current-context: kubernetes-the-hard-way
kind: Config
preferences: {}
users:
- name: admin
  user:
    client-certificate: /Users/me/kubernetes-the-hard-way-lab/certs/admin.pem
    client-key: /Users/me/kubernetes-the-hard-way-lab/certs/admin-key.pem
```

Note for me

```
With server set as --server=https://k8s-api.technoff.eu:6443, got:
Unable to connect to the server: x509: certificate is valid for kubernetes.default, not k8s-api.technoff.eu
```

## Check the health of the remote Kubernetes cluster

```
> kubectl get componentstatuses
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-2               Healthy   {"health":"true"}
etcd-1               Healthy   {"health":"true"}
etcd-0               Healthy   {"health":"true"}
```

```
> kubectl get nodes
NAME          STATUS     ROLES     AGE       VERSION
k8s-worker1   NotReady   <none>    11m       v1.10.2
k8s-worker2   NotReady   <none>    12m       v1.10.2
k8s-worker3   NotReady   <none>    12m       v1.10.2
```
