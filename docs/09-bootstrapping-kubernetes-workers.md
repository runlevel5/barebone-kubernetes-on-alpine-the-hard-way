# Bootstrapping the Kubernetes Worker Nodes

In this lab you will bootstrap three Kubernetes worker nodes. The following components will be installed on each node: [runc](https://github.com/opencontainers/runc), [container networking plugins](https://github.com/containernetworking/cni), [containerd](https://github.com/containerd/containerd), [kubelet](https://kubernetes.io/docs/admin/kubelet), and [kube-proxy](https://kubernetes.io/docs/concepts/cluster-administration/proxies).

## Prerequisites

The commands in this lab must be run on each worker instance: `kworker1`, `kworker2`, and `kworker3`.

## Provisioning a Kubernetes Worker Node

Install the OS dependencies:

```
sudo apk add socat conntrack-tools ipset
```

> The socat binary enables support for the `kubectl port-forward` command.

### Install Worker Binaries

```
sudo apk add containerd containerd-openrc kubectl kube-proxy kubelet cri-tools
```

### Configure CNI Networking

Retrieve the Pod CIDR range for the current compute instance:

```
POD_CIDR="192.168.1.0/16"
```

Create the `bridge` network configuration file:

```
sudo mkdir -p /etc/cni/net.d/
cat <<EOF | sudo tee /etc/cni/net.d/10-bridge.conf
{
    "cniVersion": "0.3.1",
    "name": "bridge",
    "type": "bridge",
    "bridge": "cnio0",
    "isGateway": true,
    "ipMasq": true,
    "ipam": {
        "type": "host-local",
        "ranges": [
          [{"subnet": "${POD_CIDR}"}]
        ],
        "routes": [{"dst": "0.0.0.0/0"}]
    }
}
EOF
```

Create the `loopback` network configuration file:

```
cat <<EOF | sudo tee /etc/cni/net.d/99-loopback.conf
{
    "cniVersion": "0.3.1",
    "name": "lo",
    "type": "loopback"
}
EOF
```

### Configure containerd

Create the `containerd` configuration file:

```
sudo mkdir -p /etc/containerd/
containerd config default | sudo tee /etc/containerd/config.toml
```

enable modules required by containerd:

```
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```

then load modules:

```
sudo modprobe overlay
sudo modprobe br_netfilter
```

setup required sysctl params, these persist across reboots:

```
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```

```
sudo sysctl -p
sudo sysctl -p /etc/sysctl.d/99-kubernetes-cri.conf
```

### Configure the Kubelet

```
sudo mkdir -p /var/lib/kubernetes/
sudo mv ${HOSTNAME}-key.pem ${HOSTNAME}.pem /var/lib/kubelet/
sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
sudo mv ca.pem /var/lib/kubernetes/
```

Create the `kubelet-config.yaml` configuration file:

```
cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.32.0.10"
podCIDR: "${POD_CIDR}"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/${HOSTNAME}.pem"
tlsPrivateKeyFile: "/var/lib/kubelet/${HOSTNAME}-key.pem"
EOF
```

Create the `/var/lib/kubelet/kubeadm-flags.env` to configure the daemon:

```
sudo mkdir -p /var/lib/kubelet

cat <<EOF | sudo tee /var/lib/kubelet/kubeadm-flags.env
export KUBELET_KUBEADM_ARGS="--config=/var/lib/kubelet/kubelet-config.yaml \\
--container-runtime=remote \\
--container-runtime-endpoint=unix:///run/containerd/containerd.sock \\
--image-pull-progress-deadline=2m \\
--kubeconfig=/var/lib/kubelet/kubeconfig \\
--cgroup-driver=cgroupfs \\
--network-plugin=cni \\
--register-node=true \\
--v=2"
EOF
```

### Configure the Kubernetes Proxy

```
sudo mv kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

Create the `kube-proxy-config.yaml` configuration file:

```
cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "10.200.0.0/16"
EOF
```

Create the `/var/lib/kubernetes/kube-proxy-flags.env` to configure the daemon:

```
sudo mkdir -p  /var/lib/kubernetes
cat <<EOF | sudo tee /var/lib/kubernetes/kube-proxy-flags.env
export KUBE_PROXY_ARGS="--config=/var/lib/kube-proxy/kube-proxy-config.yaml"
EOF
```

### Disable Swap

By default the kubelet will fail to start if swap is enabled. It is [recommended](https://github.com/kubernetes/kubernetes/issues/7294) that swap be disabled to ensure Kubernetes can provide proper resource allocation and quality of service.

If swap is enabled run the following command to disable swap immediately:

```
sudo swapoff -a
```

To ensure swap remains off after reboot, we remove swap from fstab:

```
sudo sed -i '/swap/d' /etc/fstab`
```

### Start the Worker Services

```
sudo rc-update add containerd default
sudo rc-update add kubelet default
sudo rc-update add kube-proxy default

sudo rc-service containerd start
sudo rc-service kubelet start
sudo rc-service kube-proxy start
```

> Remember to run the above commands on each worker node.

## Verification

List the registered Kubernetes nodes by running this command on the `kcontroller1` VM:

```
kubectl get nodes --kubeconfig admin.kubeconfig
```

> output

```
NAME       STATUS   ROLES    AGE   VERSION
kworker1   Ready    <none>   24s   v1.18.6
kworker2   Ready    <none>   24s   v1.18.6
kworker3   Ready    <none>   24s   v1.18.6
```

Next: [Configuring kubectl for Remote Access](10-configuring-kubectl.md)
