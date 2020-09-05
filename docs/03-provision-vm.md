# Provisioning Resources

Kubernetes requires a set of machines to host the Kubernetes control plane and the worker nodes where containers are ultimately run. In this lab you will provision the VMs required for running a secure and highly available Kubernetes cluster.

## Networking

The Kubernetes [networking model](https://kubernetes.io/docs/concepts/cluster-administration/networking/#kubernetes-model) assumes a flat network in which containers and nodes can communicate with each other. In cases where this is not desired [network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/) can limit how groups of containers are allowed to communicate with each other and external network endpoints.

> Setting up network policies is out of scope for this tutorial.

### Virtual Network

In this section a dedicated network will be setup to host the Kubernetes cluster.

Create the custom virtual network with 10.244.0.0/16 network range.

The document will not cover how to do so for a VM. Please seek documentation of the solution you choose, like VirtualBox or QEMU/KVM.

## Virtual Machine

Each VM will be provisioned with a fixed private IP address to simplify the Kubernetes bootstrapping process.

### Kubernetes Controllers

Create three VMs which will host the Kubernetes control plane.

It is recommended to have at least 10GB storage, 2GB RAM and 2 vCPU for each VM.

Next we configure the static IP address for each VM:

```
# Edit /etc/network/interfaces
# where $i is 11, 11 and 13 respectively

auto eth0
iface eth0 inet static
        hostname kcontroller${i}
        address 10.244.0.${i}
        netmask 255.255.0.0
        gateway 10.244.0.1
```

then restart networking daemon:

```
sudo rc-service networking restart
```

Next set up the hostname:

```
# where $i is 11, 12 and 13 respectively
sudo hostname kcontroller${i}
```

### Kubernetes Workers

Create three VMs which will host the Kubernetes worker nodes.

It is recommended to have at least 10GB storage, 2GB RAM and 2 vCPU for each VM.

```
# Edit /etc/network/interfaces
# where $i is 101, 102 and 103 respectively

auto eth0
iface eth0 inet static
        hostname kworker${i}
        address 10.244.0.${i}
        netmask 255.255.255.0
        gateway 10.244.0.1
```

then restart networking daemon:

```
sudo rc-service networking restart
```

### Set up /etc/hosts

Update the `/etc/hosts` of all controllers and workers with following DNS entries:

```
echo "10.244.0.11   kcontroller1" | sudo tee -a /etc/hosts
echo "10.244.0.12   kcontroller2" | sudo tee -a /etc/hosts
echo "10.244.0.13   kcontroller3" | sudo tee -a /etc/hosts
echo "10.244.0.101   kworker1" | sudo tee -a /etc/hosts
echo "10.244.0.102   kworker2" | sudo tee -a /etc/hosts
echo "10.244.0.103   kworker3" | sudo tee -a /etc/hosts
```

Next: [Provisioning a CA and Generating TLS Certificates](04-certificate-authority.md)
