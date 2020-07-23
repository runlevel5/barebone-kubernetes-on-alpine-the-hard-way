# Provisioning Resources

Kubernetes requires a set of machines to host the Kubernetes control plane and the worker nodes where containers are ultimately run. In this lab you will provision the VMs required for running a secure and highly available Kubernetes cluster.

## Networking

The Kubernetes [networking model](https://kubernetes.io/docs/concepts/cluster-administration/networking/#kubernetes-model) assumes a flat network in which containers and nodes can communicate with each other. In cases where this is not desired [network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/) can limit how groups of containers are allowed to communicate with each other and external network endpoints.

> Setting up network policies is out of scope for this tutorial.

### Virtual Network

In this section a dedicated network will be setup to host the Kubernetes cluster.

Create the custom virtual network with 172.42.42.0/24 network range.

The document will not cover how to do so for a VM. Please seek documentation of the solution you choose, like VirtualBox or QEMU/KVM.

## Virtual Machine

Each VM will be provisioned with a fixed private IP address to simplify the Kubernetes bootstrapping process.

### Kubernetes Controllers

Create three VMs which will host the Kubernetes control plane.

It is recommended to have at least 10GB storage, 2GB RAM and 2 vCPU for each VM.

Next we configure the static IP address for each VM:

```
# Edit /etc/network/interfaces
# where $i is 101, 102 and 103 respectively

auto eth0
iface eth0 inet static
        hostname kcontroller${i}
        address 172.42.42.${i}
        netmask 255.255.255.0
        gateway 172.42.42.1
```

then restart networking daemon:

```
sudo rc-service networking restart
```

Next set up the hostname:

```
# where $i is 101, 102 and 103 respectively
sudo hostname kcontroller${i}
```

and update the hosts file:

```
echo "172.42.42.101   kcontroller1" | sudo tee -a /etc/hosts
echo "172.42.42.102   kcontroller2" | sudo tee -a /etc/hosts
echo "172.42.42.103   kcontroller3" | sudo tee -a /etc/hosts
echo "172.42.42.201   kworker1" | sudo tee -a /etc/hosts
echo "172.42.42.202   kworker2" | sudo tee -a /etc/hosts
echo "172.42.42.203   kworker3" | sudo tee -a /etc/hosts
```

### Kubernetes Workers

Create three VMs which will host the Kubernetes worker nodes.

It is recommended to have at least 10GB storage, 2GB RAM and 2 vCPU for each VM.

```
# Edit /etc/network/interfaces
# where $i is 201, 202 and 203 respectively

auto eth0
iface eth0 inet static
        hostname kworker${i}
        address 172.42.42.${i}
        netmask 255.255.255.0
        gateway 172.42.42.1
```

then restart networking daemon:

```
sudo rc-service networking restart
```

Next set up the hostname:

```
# where $i is 201, 202 and 203 respectively
sudo hostname kworker${i}
```

and update the hosts file:

```
echo "172.42.42.101   kcontroller1" | sudo tee -a /etc/hosts
echo "172.42.42.102   kcontroller2" | sudo tee -a /etc/hosts
echo "172.42.42.103   kcontroller2" | sudo tee -a /etc/hosts
echo "172.42.42.201   kworker1" | sudo tee -a /etc/hosts
echo "172.42.42.202   kworker2" | sudo tee -a /etc/hosts
echo "172.42.42.203   kworker3" | sudo tee -a /etc/hosts
```

Next: [Provisioning a CA and Generating TLS Certificates](04-certificate-authority.md)
