# Provisioning a CA and Generating TLS Certificates 

## Certificate Authority

### Create the CA configuration file

```
cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "26280h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "26280h"
      }
    }
  }
}
EOF
```

### Create the CA cert request

```
{
cat > ca-csr.json <<EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "LV",
      "L": "Daugavpils",
      "O": "Kubernetes",
      "OU": "CA"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca
}
```

Result

```
ca.pem
ca-key.pem
```

## Client and Server Certificates

### Create the admin client certificate signing request

```
{
cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "LV",
      "L": "Daugavpils",
      "O": "system:masters",
      "OU": "Kubernetes The Hard Way"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin
}
```

Result

```
admin-key.pem
admin.pem
```

## Kubelet Client Certificates 

### Generate a certificate and private key pair for each worker node

```
{
for i in k8s-worker1 k8s-worker2 k8s-worker3; do
cat > ${i}-csr.json <<EOF
{
  "CN": "system:node:${i}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "LV",
      "L": "Daugavpils",
      "O": "system:nodes",
      "OU": "Kubernetes The Hard Way"
    }
  ]
}
EOF

IP=`nslookup ${i} | awk '/^Address: / { print $2 }'`

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${i},$IP \
  -profile=kubernetes \
  ${i}-csr.json | cfssljson -bare ${i}
done
}
```

Result

```
k8s-worker1.pem
k8s-worker1-key.pem
k8s-worker2.pem
k8s-worker2-key.pem
k8s-worker3.pem
k8s-worker3-key.pem
```

## Controller Manager Client Certificate

```
{
cat > kube-controller-manager-csr.json <<EOF
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "LV",
      "L": "Daugavpils",
      "O": "system:kube-controller-manager",
      "OU": "Kubernetes The Hard Way"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager
}
```

Result

```
kube-controller-manager-key.pem
kube-controller-manager.pem
```

## Generate the kube-proxy certificate

```
{
cat > kube-proxy-csr.json <<EOF
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "LV",
      "L": "Daugavpils",
      "O": "system:node-proxier",
      "OU": "Kubernetes The Hard Way"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-proxy-csr.json | cfssljson -bare kube-proxy
}
```

Result

```
kube-proxy-key.pem
kube-proxy.pem
```

## Scheduler Client Certificate

```
{
cat > kube-scheduler-csr.json <<EOF
{
  "CN": "system:kube-scheduler",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "LV",
      "L": "Daugavpils",
      "O": "system:kube-scheduler",
      "OU": "Kubernetes The Hard Way"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-scheduler-csr.json | cfssljson -bare kube-scheduler
}
```

Result

```
kube-scheduler-key.pem
kube-scheduler.pem
```

## Kubernetes API Server certificate

```
10.98.95.11/12/13 -> the 3 k8s control plane hosts
10.98.95.18 -> the Kube-api load balancer
10.98.96.1 -> the Kube-api ("internal") service IP
```

```
{
cat > kubernetes-csr.json <<EOF
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "LV",
      "L": "Daugavpils",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=10.98.95.11,10.98.95.12,10.98.95.13,10.98.95.18,10.98.96.1,127.0.0.1,kubernetes.default \
  -profile=kubernetes \
  kubernetes-csr.json | cfssljson -bare kubernetes
}
```

Result

```
kubernetes-key.pem
kubernetes.pem
```

## Service Account Key Pair

```
{
cat > service-account-csr.json <<EOF
{
  "CN": "service-accounts",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "LV",
      "L": "Daugavpils",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  service-account-csr.json | cfssljson -bare service-account
}
```

Result

```
service-account-key.pem
service-account.pem
```

## Copy the generated certificates/keys to hosts

```
for i in k8s-controller1 k8s-controller2 k8s-controller3; do
scp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
  service-account-key.pem service-account.pem defo@${i}:~/
done

for i in k8s-worker1 k8s-worker2 k8s-worker3; do
scp ca.pem ${i}-key.pem ${i}.pem defo@${i}:~/
done
```





