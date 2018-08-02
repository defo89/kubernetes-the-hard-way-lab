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

Grant cluster wide permissions for Helm's Tiller.
https://docs.gitlab.com/ee/install/kubernetes/preparation/tiller.html

```
kubectl delete -f manifests/rbac-fix.yaml
kubectl apply -f manifests/rbac-fix.yaml
helm init --upgrade --service-account tiller
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
kubectl apply -f manifests/metallb-config.yaml
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

https://coreos.com/operators/prometheus/docs/latest/

### Installation

```
helm repo add coreos https://s3-eu-west-1.amazonaws.com/coreos-charts/stable/
helm install coreos/prometheus-operator --name prometheus-operator --namespace monitoring
```

Output

```
NAME:   prometheus-operator
LAST DEPLOYED: Fri Jul  6 17:14:50 2018
NAMESPACE: monitoring
STATUS: DEPLOYED

RESOURCES:
==> v1/Pod(related)
NAME                                READY  STATUS   RESTARTS  AGE
prometheus-operator-d75587d6-kbx77  1/1    Running  0         1m

==> v1/ConfigMap
NAME                 DATA  AGE
prometheus-operator  1     1m

==> v1/ServiceAccount
NAME                 SECRETS  AGE
prometheus-operator  1        1m

==> v1beta1/ClusterRole
NAME                     AGE
prometheus-operator      1m
psp-prometheus-operator  1m

==> v1beta1/ClusterRoleBinding
NAME                     AGE
prometheus-operator      1m
psp-prometheus-operator  1m

==> v1beta1/Deployment
NAME                 DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
prometheus-operator  1        1        1           1          1m

==> v1beta1/PodSecurityPolicy
NAME                 DATA   CAPS      SELINUX   RUNASUSER  FSGROUP    SUPGROUP  READONLYROOTFS  VOLUMES
prometheus-operator  false  RunAsAny  RunAsAny  MustRunAs  MustRunAs  false     configMap,emptyDir,projected,secret,downwardAPI,persistentVolumeClaim


NOTES:
The Prometheus Operator has been installed. Check its status by running:
  kubectl --namespace monitoring get pods -l "app=prometheus-operator,release=prometheus-operator"

Visit https://github.com/coreos/prometheus-operator for instructions on how
to create & configure Alertmanager
```

```
helm install coreos/kube-prometheus --name kube-prometheus --set rbacEnable=true --namespace monitoring
```

Output

```
NAME:   kube-prometheus
LAST DEPLOYED: Fri Jul  6 17:19:35 2018
NAMESPACE: monitoring
STATUS: DEPLOYED

RESOURCES:
==> v1/Service
NAME                                              TYPE       CLUSTER-IP    EXTERNAL-IP  PORT(S)              AGE
kube-prometheus-alertmanager                      ClusterIP  10.98.96.20   <none>       9093/TCP             3s
kube-prometheus-exporter-kube-controller-manager  ClusterIP  None          <none>       10252/TCP            3s
kube-prometheus-exporter-kube-dns                 ClusterIP  None          <none>       10054/TCP,10055/TCP  2s
kube-prometheus-exporter-kube-etcd                ClusterIP  None          <none>       4001/TCP             2s
kube-prometheus-exporter-kube-scheduler           ClusterIP  None          <none>       10251/TCP            2s
kube-prometheus-exporter-kube-state               ClusterIP  10.98.96.162  <none>       80/TCP               2s
kube-prometheus-exporter-node                     ClusterIP  10.98.96.84   <none>       9100/TCP             2s
kube-prometheus-grafana                           ClusterIP  10.98.96.58   <none>       80/TCP               2s
kube-prometheus                                   ClusterIP  10.98.96.188  <none>       9090/TCP             2s

==> v1beta1/DaemonSet
NAME                           DESIRED  CURRENT  READY  UP-TO-DATE  AVAILABLE  NODE SELECTOR  AGE
kube-prometheus-exporter-node  3        3        0      3           0          <none>         2s

==> v1beta1/Deployment
NAME                                 DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
kube-prometheus-exporter-kube-state  1        1        1           0          2s
kube-prometheus-grafana              1        1        1           0          2s

==> v1beta1/PodSecurityPolicy
NAME                                 DATA   CAPS      SELINUX   RUNASUSER  FSGROUP    SUPGROUP  READONLYROOTFS  VOLUMES
kube-prometheus-alertmanager         false  RunAsAny  RunAsAny  MustRunAs  MustRunAs  false     configMap,emptyDir,projected,secret,downwardAPI,persistentVolumeClaim
kube-prometheus-exporter-kube-state  false  RunAsAny  RunAsAny  MustRunAs  MustRunAs  false     configMap,emptyDir,projected,secret,downwardAPI,persistentVolumeClaim
kube-prometheus-exporter-node        false  RunAsAny  RunAsAny  MustRunAs  MustRunAs  false     configMap,emptyDir,projected,secret,downwardAPI,persistentVolumeClaim,hostPath
kube-prometheus-grafana              false  RunAsAny  RunAsAny  MustRunAs  MustRunAs  false     configMap,emptyDir,projected,secret,downwardAPI,persistentVolumeClaim,hostPath
kube-prometheus                      false  RunAsAny  RunAsAny  MustRunAs  MustRunAs  false     configMap,emptyDir,projected,secret,downwardAPI,persistentVolumeClaim

==> v1/ServiceAccount
NAME                                 SECRETS  AGE
kube-prometheus-exporter-kube-state  1        3s
kube-prometheus-exporter-node        1        3s
kube-prometheus-grafana              1        3s
kube-prometheus                      1        3s

==> v1beta1/ClusterRoleBinding
NAME                                     AGE
psp-kube-prometheus-alertmanager         3s
kube-prometheus-exporter-kube-state      3s
psp-kube-prometheus-exporter-kube-state  3s
psp-kube-prometheus-exporter-node        3s
psp-kube-prometheus-grafana              3s
kube-prometheus                          3s
psp-kube-prometheus                      3s

==> v1/Prometheus
NAME             AGE
kube-prometheus  2s

==> v1/Pod(related)
NAME                                                  READY  STATUS             RESTARTS  AGE
kube-prometheus-exporter-node-26sqx                   0/1    ContainerCreating  0         2s
kube-prometheus-exporter-node-4rsqt                   0/1    ContainerCreating  0         2s
kube-prometheus-exporter-node-bmqj6                   0/1    ContainerCreating  0         2s
kube-prometheus-exporter-kube-state-844bb6f589-crbjb  0/2    ContainerCreating  0         2s
kube-prometheus-grafana-57d5b4d79f-nmz99              0/2    ContainerCreating  0         2s

==> v1beta1/ClusterRole
NAME                                     AGE
psp-kube-prometheus-alertmanager         3s
kube-prometheus-exporter-kube-state      3s
psp-kube-prometheus-exporter-kube-state  3s
psp-kube-prometheus-exporter-node        3s
psp-kube-prometheus-grafana              3s
kube-prometheus                          3s
psp-kube-prometheus                      3s

==> v1beta1/Role
kube-prometheus-exporter-kube-state  3s

==> v1/Alertmanager
kube-prometheus  2s

==> v1/Secret
NAME                          TYPE    DATA  AGE
alertmanager-kube-prometheus  Opaque  1     3s
kube-prometheus-grafana       Opaque  2     3s

==> v1/ConfigMap
NAME                                              DATA  AGE
kube-prometheus-alertmanager                      1     3s
kube-prometheus-exporter-kube-controller-manager  1     3s
kube-prometheus-exporter-kube-etcd                1     3s
kube-prometheus-exporter-kube-scheduler           1     3s
kube-prometheus-exporter-kube-state               1     3s
kube-prometheus-exporter-kubelets                 1     3s
kube-prometheus-exporter-kubernetes               1     3s
kube-prometheus-exporter-node                     1     3s
kube-prometheus-grafana                           10    3s
kube-prometheus-rules                             1     3s
kube-prometheus                                   1     3s

==> v1beta1/RoleBinding
NAME                                 AGE
kube-prometheus-exporter-kube-state  3s

==> v1/ServiceMonitor
NAME                                              AGE
kube-prometheus-alertmanager                      2s
kube-prometheus-exporter-kube-controller-manager  2s
kube-prometheus-exporter-kube-dns                 2s
kube-prometheus-exporter-kube-etcd                2s
kube-prometheus-exporter-kube-scheduler           2s
kube-prometheus-exporter-kube-state               2s
kube-prometheus-exporter-kubelets                 2s
kube-prometheus-exporter-kubernetes               2s
kube-prometheus-exporter-node                     2s
kube-prometheus-grafana                           2s
kube-prometheus                                   2s


NOTES:
DEPRECATION NOTICE:

- alertmanager.ingress.fqdn is not used anymore, use alertmanager.ingress.hosts []
- prometheus.ingress.fqdn is not used anymore, use prometheus.ingress.hosts []
- grafana.ingress.fqdn is not used anymore, use prometheus.grafana.hosts []
```

External access to Grafana using MetalLB

```
kubectl apply -n monitoring -f manifests/lb-monitoring.yaml
```

### Verification

```
kubectl get pods  -n monitoring
```

