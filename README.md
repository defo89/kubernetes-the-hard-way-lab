# Kubernetes The Hard Way in Lab (Bare Metal)

## Design
| Name | IP address | Role |
| ---|---|---|
| networker	| 10.98.95.230 | DHCP and DNS server |
| haproxy1	| 10.98.95.18 | Load Balancer for k8s API |
| k8s-controller1 | 10.98.95.11 | controller node |
| k8s-controller2 | 10.98.95.12 | controller node |
| k8s-controller3 | 10.98.95.13 | controller node |
| k8s-worker1 | 10.98.95.21 | worker node |
| k8s-worker2 | 10.98.95.22 | worker node |
| k8s-worker3 | 10.98.95.23 | worker node |


| Network / IP | Description |
| -------------|-------------|
| 10.98.95.0/24 | LAN (technoff.eu) |
| 10.98.100.0/22 | k8s Pod network |
| 10.98.96.0/24 | k8s Service network |
| 10.98.95.18 | k8s API server (external) |
| 10.98.96.1 | k8s API server (internal) |
| 10.98.96.10	| k8s DNS |

Setup that worked in my environment:
- Operating System: Ubuntu 18.04 LTS (VMs on top of VMware ESXi)
- Networking solution: Weave Net
- DNS Add-on - coredns

Gotchas:
