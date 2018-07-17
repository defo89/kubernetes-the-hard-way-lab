# Configuring storage access

Configuring storage for Persistent Volumes.

https://docs.openshift.com/enterprise/3.1/install_config/persistent_storage/persistent_storage_nfs.html
https://docs.openshift.org/latest/install_config/storage_examples/shared_storage.html
https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/

## Prepare Worker Nodes

Commands will be run on each controller instance: k8s-worker1, k8s-worker2, k8s-worker3.

Install NFS common tools.

```
sudo apt-get install -y nfs-common
```


## Create and apply persistent volume

```
kubectl apply -f manifests/pv-volume.yaml
```

Output

```
persistentvolume "pv0001" created
```

```
> kubectl describe pv pv0001
Name:            pv0001
Labels:          <none>
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"PersistentVolume","metadata":{"annotations":{},"name":"pv0001","namespace":""},"spec":{"accessModes":["ReadWriteMany"],"capa...
                 pv.kubernetes.io/bound-by-controller=yes
StorageClass:    slow
Status:          Bound
Claim:           default/pvc0001
Reclaim Policy:  Delete
Access Modes:    RWX
Capacity:        2Gi
Message:
Source:
    Type:      NFS (an NFS mount that lasts the lifetime of a pod)
    Server:    media
    Path:      /mnt/nfs/pv0001
    ReadOnly:  false
Events:        <none>
```

## Create and apply persistent volume claim

```
kubectl apply -f manifests/pv-claim.yaml
```

Output

```
persistentvolumeclaim "task-pv-claim" created
```

```
> kubectl describe pvc pvc0001
Name:          pvc0001
Namespace:     default
StorageClass:  slow
Status:        Bound
Volume:        pv0001
Labels:        <none>
Annotations:   kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"PersistentVolumeClaim","metadata":{"annotations":{},"name":"pvc0001","namespace":"default"},"spec":{"accessModes":["ReadWrit...
               pv.kubernetes.io/bind-completed=yes
               pv.kubernetes.io/bound-by-controller=yes
Capacity:      2Gi
Access Modes:  RWX
Events:        <none>
```

## Create pod that uses your PersistentVolumeClaim as a volume

```
kubectl apply -f manifests/nginx-deployment-pv.yaml
```

Output

```
service "nginx-pv" created
deployment "nginx-deployment-pv" created
```