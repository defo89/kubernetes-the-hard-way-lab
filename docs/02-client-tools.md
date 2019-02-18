# Installing the Client Tools (OSX)

## cfssl

```
> brew install cfssl
> cfssl version
Version: 1.3.2
Revision: dev
Runtime: go1.10.2
```

## kubectl

```
> brew install kubernetes-cli
> kubectl version --client
Client Version: version.Info{Major:"1", Minor:"8", GitVersion:"v1.8.6", GitCommit:"6260bb08c46c31eea6cb538b34a9ceb3e406689c", GitTreeState:"clean", BuildDate:"2017-12-21T06:34:11Z", GoVersion:"go1.8.3", Compiler:"gc", Platform:"darwin/amd64"}
```

## Copy public key to all nodes

```
for i in k8s-controller1 k8s-controller2 k8s-controller3; do
    ssh-copy-id defo@${i}
done

for i in k8s-worker1 k8s-worker2 k8s-worker3; do
    ssh-copy-id defo@${i}
done
```
