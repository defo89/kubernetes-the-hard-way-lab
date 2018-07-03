# LOAD BALANCER

## Install HAProxy

```
sudo apt-get update && \
sudo apt-get -y upgrade && \
sudo apt-get -y install haproxy=1.8.\*
```

## Configure Loab Balancing for k8s API

Edit file /etc/haproxy/haproxy.cfg (append to end)

```
backend k8s-api
  mode tcp
  timeout server 1h
  option tcplog
  option tcp-check
  balance roundrobin
  default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
  server k8s-api-1 10.98.95.11:6443 check
  server k8s-api-2 10.98.95.12:6443 check
  server k8s-api-3 10.98.95.13:6443 check

frontend k8s-api
  bind 0.0.0.0:6443
  mode tcp
  option tcplog
  timeout client 1h
  default_backend k8s-api
```

```
systemctl enable haproxy
systemctl start haproxy
```

Check the status

```
systemctl status haproxy
```