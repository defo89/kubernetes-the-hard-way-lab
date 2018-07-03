# Notes from Troubleshooting and Operations

## Changing Worker certificates

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
2018/07/02 11:16:32 [INFO] generate received request
2018/07/02 11:16:32 [INFO] received CSR
2018/07/02 11:16:32 [INFO] generating key: rsa-2048
2018/07/02 11:16:32 [INFO] encoded CSR
2018/07/02 11:16:32 [INFO] signed certificate with serial number 
2018/07/02 11:16:32 [INFO] generate received request
2018/07/02 11:16:32 [INFO] received CSR
2018/07/02 11:16:32 [INFO] generating key: rsa-2048
2018/07/02 11:16:32 [INFO] encoded CSR
2018/07/02 11:16:32 [INFO] signed certificate with serial number 
2018/07/02 11:16:33 [INFO] generate received request
2018/07/02 11:16:33 [INFO] received CSR
2018/07/02 11:16:33 [INFO] generating key: rsa-2048
2018/07/02 11:16:33 [INFO] encoded CSR
2018/07/02 11:16:33 [INFO] signed certificate with serial number 
```

```
for i in k8s-worker1 k8s-worker2 k8s-worker3; do
scp ${i}-key.pem ${i}.pem defo@${i}:~/
done
```

```
{
  HOSTNAME=`hostname -s`
  sudo systemctl stop kubelet
  sudo mv ${HOSTNAME}-key.pem ${HOSTNAME}.pem /var/lib/kubelet/
}
```

```
{
  sudo systemctl daemon-reload
  sudo systemctl start kubelet
  sudo systemctl status kubelet --no-pager
}
```

## Disable swap on Workers

```
> systemctl status kubelet.service 
● kubelet.service - Kubernetes Kubelet.service; enabled; vendor preset: enabled)
   Loaded: loaded (/etc/systemd/system/kubelet.service; enabled; vendor preset: enabled)
   Active: activating (auto-restart) (Result: exit-code) since Mon 2018-07-02 11:00:47 UTC; 
     Docs: https://github.com/kubernetes/kubernetes
  Process: 23375 ExecStart=/usr/local/bin/kubelet --config=/var/lib/kubelet/kubelet-config.y
 Main PID: 23375 (code=exited, status=255)

Jul 02 11:03:18 k8s-worker2 kubelet[24023]: F0702 11:03:18.505516   24023 server.go:233] failed to run Kubelet: Running with swap on is not supported, please disable swap! or set --fail-swap-on flag to false. /proc/swaps contained:
Jul 02 11:03:18 k8s-worker2 systemd[1]: kubelet.service: Main process exited, code=exited, s
Jul 02 11:03:18 k8s-worker2 systemd[1]: kubelet.service: Failed with result 'exit-code'.
```

Disable swap for current session.

```
> free -h
              total        used        free      shared  buff/cache   available
Mem:           985M        144M        236M        1.1M        604M        688M
Swap:          1.9G          0B        1.9G
```

```
> sudo swapoff -a 
> free -h         
              total        used        free      shared  buff/cache   available
Mem:           985M        167M        195M        1.1M        623M        663M
Swap:            0B          0B          0B
```

Comment out swap in /etc/fstab file.

```
> 
> sudo vim /etc/fstab
#/swap.img      none    swap    sw      0       0
> sudo shutdown -r now
```