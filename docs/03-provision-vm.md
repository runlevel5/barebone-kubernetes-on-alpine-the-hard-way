# Provisioning Resources

Kubernetes requires a set of machines to host the Kubernetes control plane and the worker nodes where containers are ultimately run. In this lab you will provision the VMs required for running a secure and highly available Kubernetes cluster.

## Networking

The Kubernetes [networking model](https://kubernetes.io/docs/concepts/cluster-administration/networking/#kubernetes-model) assumes a flat network in which containers and nodes can communicate with each other. In cases where this is not desired [network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/) can limit how groups of containers are allowed to communicate with each other and external network endpoints.

> Setting up network policies is out of scope for this tutorial.

### Virtual Network

In this section a dedicated network will be setup to host the Kubernetes cluster.

Create the custom virtual network with 10.240.0.0/24 network range.

The document will not cover how to do so for a VM. Please seek documentation of the solution you choose, like VirtualBox or QEMU/KVM.

> The `10.240.0.0/24` IP address range can host up to 254 compute instances.

## Virtual Machine

The virtual machines in this lab will be provisioned using [Alpine (virt variant)](https://alpinelinux.org/downloads/) 3.12, which has good support for the [containerd container runtime](https://github.com/containerd/containerd). Each compute instance will be provisioned with a fixed private IP address to simplify the Kubernetes bootstrapping process.

At the time of the writing, Alpine edge has all the up-to-date kubernetes packages. Please enable Alpine edge repositories:

```sh
echo "http://dl-cdn.alpinelinux.org/alpine/edge/main" | sudo tee /etc/apk/repositories
echo "http://dl-cdn.alpinelinux.org/alpine/edge/community" | sudo tee -a /etc/apk/repositories
echo "http://dl-cdn.alpinelinux.org/alpine/edge/testing" | sudo tee -a /etc/apk/repositories

sudo apk update
sudo apk upgrade
```

### Kubernetes Controllers

Create three VMs which will host the Kubernetes control plane.

It is recommended to have at least 10GB storage, 2GB RAM and 2 vCPU for each VM.

Next we configure the static IP address for each VM:

```
# Edit /etc/network/interfaces
# where $i is 100, 101 and 103 respectively

auto eth0
iface eth0 inet static
        address 10.240.0.${i}
        netmask 255.255.255.0
        gateway 10.240.0.1
```

then restart networking daemon:

```
sudo rc-service networking restart
```

### Kubernetes Workers

Create three VMs which will host the Kubernetes worker nodes.

It is recommended to have at least 10GB storage, 2GB RAM and 2 vCPU for each VM.

```
# Edit /etc/network/interfaces
# where $i is 201, 202 and 203 respectively

auto eth0
iface eth0 inet static
        address 10.240.0.${i}
        netmask 255.255.255.0
        gateway 10.240.0.1
```

then restart networking daemon:

```
sudo rc-service networking restart
```

Next: [Provisioning a CA and Generating TLS Certificates](04-certificate-authority.md)
