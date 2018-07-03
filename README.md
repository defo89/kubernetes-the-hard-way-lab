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
| 10.98.95.18 | k8s API server (external IP - haproxy) |
| 10.98.96.1 | k8s API server (internal IP) |
| 10.98.96.10	| k8s DNS |

## Building Blocks

Setup that worked in my environment:
- Operating System: Ubuntu 18.04 LTS (VMs on top of VMware ESXi)
- Networking solution: Weave Net
- DNS Add-on - coredns

## Labs

* [Infrastructure Setup](docs/01-infrastructure-setup.md)
* [Installing the Client Tools](docs/02-client-tools.md)
* [Provisioning a CA and Generating TLS Certificates](docs/03-certificate-authority.md)
* [Kubernetes Configuration Files for Authentication](docs/04-kubernetes-configuration-files.md)
* [Generating the Data Encryption Config and Key](docs/05-data-encryption-keys.md)
* [Bootstrapping the etcd Cluster](docs/06-bootstrapping-etcd.md)
* [Load Balancer](docs/07-kubernetes-api-lb.md)
* [Bootstrapping the Kubernetes Control Plane](docs/08-bootstrapping-kubernetes-controllers.md)
* [Bootstrapping the Kubernetes Worker Nodes](docs/09-bootstrapping-kubernetes-workers.md)
* [Configuring kubectl for Remote Access](docs/10-configuring-kubectl.md)
* [Configure Kubernetes Networking](docs/11-configuring-networking.md)
* [Deploying the DNS Cluster Add-on](docs/12-dns-addon.md)
* [Testing and Playing with Kubernetes](docs/13-playing-with-k8s.md)
* [Verification and Troubleshooting](docs/98-commands.md)
* [Notes from Troubleshooting and Operations](docs/99-scenario-notes.md)

## Credits

* [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)
* [ON-PREM K8S](https://blog.csnet.me/k8s-thw/)