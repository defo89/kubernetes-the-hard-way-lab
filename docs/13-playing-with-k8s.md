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
helm install coreos/kube-prometheus --name kube-prometheus --namespace monitoring
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

## Gitlab

```
helm repo add gitlab https://charts.gitlab.io/
helm repo update
helm upgrade --install gitlab gitlab/gitlab \
  --timeout 600 \
  --set global.hosts.domain=technoff.eu \
  --set global.hosts.externalIP=10.98.95.5 \
  --set certmanager-issuer.email=dmitri@fedo.tf
  --set gitlab.migrations.image.repository=registry.gitlab.com/gitlab-org/build/cng/gitlab-rails-ce
  --set gitlab.sidekiq.image.repository=registry.gitlab.com/gitlab-org/build/cng/gitlab-sidekiq-ce
  --set gitlab.unicorn.image.repository=registry.gitlab.com/gitlab-org/build/cng/gitlab-unicorn-ce
```

Output

```
Release "gitlab" does not exist. Installing it now.
NAME:   gitlab
LAST DEPLOYED: Fri Jul  6 17:23:45 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/PersistentVolumeClaim
NAME                      STATUS   VOLUME  CAPACITY  ACCESS MODES  STORAGECLASS  AGE
gitlab-minio              Pending  5s
gitlab-postgresql         Pending  5s
gitlab-prometheus-server  Pending  5s
gitlab-redis              Pending  5s

==> v1beta1/RoleBinding
NAME                  AGE
gitlab-gitlab-runner  5s
gitlab-nginx-ingress  5s

==> v1beta1/Deployment
NAME                                  DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
certmanager-gitlab                    1        1        1           0          4s
gitlab-gitlab-runner                  1        1        1           0          4s
gitlab-gitlab-shell                   1        1        1           0          4s
gitlab-sidekiq-all-in-1               1        1        1           0          4s
gitlab-task-runner                    1        1        1           0          4s
gitlab-unicorn                        1        1        1           0          4s
gitlab-minio                          1        1        1           0          4s
gitlab-nginx-ingress-controller       3        3        3           0          4s
gitlab-nginx-ingress-default-backend  2        2        2           0          4s
gitlab-postgresql                     1        1        1           0          4s
gitlab-prometheus-server              1        1        1           0          4s
gitlab-redis                          1        1        1           0          4s
gitlab-registry                       1        1        1           0          4s

==> v1/Pod(related)
NAME                                                   READY  STATUS             RESTARTS  AGE
certmanager-gitlab-54467869c4-lmvdk                    0/2    ContainerCreating  0         4s
gitlab-gitlab-runner-5b74b9b46f-wx5k2                  0/1    Init:0/1           0         3s
gitlab-gitlab-shell-747b787dcf-gml5g                   0/1    Init:0/1           0         4s
gitlab-sidekiq-all-in-1-796769d649-tcxj9               0/1    Pending            0         3s
gitlab-task-runner-7f8db97fc5-mscsn                    0/1    Init:0/1           0         3s
gitlab-unicorn-5b7cbb964-gdm5n                         0/1    Pending            0         3s
gitlab-minio-6cb66d6f9f-lkcdt                          0/1    Pending            0         3s
gitlab-nginx-ingress-controller-6658b5796b-fm768       0/1    ContainerCreating  0         3s
gitlab-nginx-ingress-controller-6658b5796b-p2gqt       0/1    ContainerCreating  0         3s
gitlab-nginx-ingress-controller-6658b5796b-tt79q       0/1    ContainerCreating  0         3s
gitlab-nginx-ingress-default-backend-699b9476dd-65q9g  0/1    ContainerCreating  0         3s
gitlab-nginx-ingress-default-backend-699b9476dd-ckhkh  0/1    ContainerCreating  0         3s
gitlab-postgresql-5578b89f58-26ss7                     0/2    Pending            0         3s
gitlab-prometheus-server-847c8bb76-vxpdc               0/2    Pending            0         3s
gitlab-redis-b9bf85566-7nh84                           0/2    Pending            0         3s
gitlab-registry-77bc9d6f9f-h8cbd                       0/1    Init:0/1           0         2s
gitlab-gitaly-0                                        0/1    Pending            0         3s
gitlab-issuer.1-qjw58                                  0/1    ContainerCreating  0         4s
gitlab-migrations.1-2crcf                              0/1    Init:0/1           0         4s
gitlab-minio-create-buckets.1-gzt9b                    0/1    ContainerCreating  0         4s

==> v1/ServiceAccount
NAME                                  SECRETS  AGE
gitlab-certmanager-issuer-admin       1        5s
certmanager-gitlab                    1        5s
gitlab-gitlab-runner                  1        5s
gitlab-nginx-ingress                  1        5s
gitlab-prometheus-alertmanager        1        5s
gitlab-prometheus-kube-state-metrics  1        5s
gitlab-prometheus-node-exporter       1        5s
gitlab-prometheus-server              1        5s

==> v1beta1/CustomResourceDefinition
NAME                               AGE
certificates.certmanager.k8s.io    5s
clusterissuers.certmanager.k8s.io  5s
issuers.certmanager.k8s.io         5s

==> v1beta1/ClusterRole
certmanager-gitlab                    5s
gitlab-nginx-ingress                  5s
gitlab-prometheus-kube-state-metrics  5s
gitlab-prometheus-server              5s

==> v1beta1/ClusterRoleBinding
NAME                                  AGE
gitlab-certmanager-issuer-admin       5s
certmanager-gitlab                    5s
gitlab-nginx-ingress                  5s
gitlab-prometheus-alertmanager        5s
gitlab-prometheus-kube-state-metrics  5s
gitlab-prometheus-node-exporter       5s
gitlab-prometheus-server              5s

==> v1beta1/Role
NAME                  AGE
gitlab-gitlab-runner  5s
gitlab-nginx-ingress  5s

==> v1beta2/StatefulSet
NAME           DESIRED  CURRENT  AGE
gitlab-gitaly  1        1        4s

==> v1/Job
NAME                           DESIRED  SUCCESSFUL  AGE
gitlab-issuer.1                1        0           4s
gitlab-migrations.1            1        0           4s
gitlab-minio-create-buckets.1  1        0           4s

==> v1beta1/Ingress
NAME             HOSTS                 ADDRESS  PORTS  AGE
gitlab-unicorn   gitlab.technoff.eu    80, 443  4s
gitlab-minio     minio.technoff.eu     80, 443  4s
gitlab-registry  registry.technoff.eu  80, 443  4s

==> v1/Service
NAME                                  TYPE          CLUSTER-IP    EXTERNAL-IP  PORT(S)                                  AGE
gitlab-gitaly                         ClusterIP     None          <none>       8075/TCP,9236/TCP                        4s
gitlab-gitlab-shell                   ClusterIP     10.98.96.252  <none>       22/TCP                                   4s
gitlab-unicorn                        ClusterIP     10.98.96.184  <none>       8080/TCP,8181/TCP                        4s
gitlab-minio-svc                      ClusterIP     10.98.96.86   <none>       9000/TCP                                 4s
gitlab-nginx-ingress-controller       LoadBalancer  10.98.96.210  <pending>    80:31940/TCP,443:30311/TCP,22:30480/TCP  4s
gitlab-nginx-ingress-default-backend  ClusterIP     10.98.96.139  <none>       80/TCP                                   4s
gitlab-postgresql                     ClusterIP     10.98.96.113  <none>       5432/TCP                                 4s
gitlab-prometheus-server              ClusterIP     10.98.96.8    <none>       80/TCP                                   4s
gitlab-redis                          ClusterIP     10.98.96.53   <none>       6379/TCP,9121/TCP                        4s
gitlab-registry                       ClusterIP     10.98.96.227  <none>       5000/TCP                                 4s

==> v1beta1/PodDisruptionBudget
NAME                                  MIN AVAILABLE  MAX UNAVAILABLE  ALLOWED DISRUPTIONS  AGE
gitlab-gitaly                         N/A            1                0                    3s
gitlab-gitlab-shell                   N/A            1                0                    3s
gitlab-sidekiq                        N/A            1                0                    3s
gitlab-unicorn                        N/A            1                0                    3s
gitlab-minio-v1                       N/A            1                0                    3s
gitlab-nginx-ingress-controller       2              N/A              0                    3s
gitlab-nginx-ingress-default-backend  1              N/A              0                    3s
gitlab-redis-v1                       N/A            1                0                    3s
gitlab-registry-v1                    N/A            1                0                    3s

==> v1/ConfigMap
NAME                                   DATA  AGE
gitlab-certmanager-issuer-certmanager  2     6s
gitlab-gitlab-runner                   3     6s
gitlab-gitaly                          3     6s
gitlab-gitlab-shell                    2     6s
gitlab-nginx-ingress-tcp               1     6s
gitlab-migrations                      4     6s
gitlab-sidekiq-all-in-1                1     6s
gitlab-sidekiq                         5     6s
gitlab-task-runner                     4     6s
gitlab-unicorn                         7     6s
gitlab-minio-config-cm                 3     6s
gitlab-nginx-ingress-controller        7     6s
gitlab-postgresql                      0     6s
gitlab-prometheus-server               3     6s
gitlab-redis                           2     6s
gitlab-registry                        2     6s

==> v2beta1/HorizontalPodAutoscaler
NAME                     REFERENCE                           TARGETS        MINPODS  MAXPODS  REPLICAS  AGE
gitlab-gitlab-shell      Deployment/gitlab-gitlab-shell      <unknown>/75%  2        10       0         4s
gitlab-sidekiq-all-in-1  Deployment/gitlab-sidekiq-all-in-1  <unknown>/75%  1        10       0         3s
gitlab-unicorn           Deployment/gitlab-unicorn           <unknown>/75%  2        10       0         3s
gitlab-registry          Deployment/gitlab-registry          <unknown>/75%  2        10       0         3s
```

