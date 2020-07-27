# Provisioning Pod Network Routes

Pods scheduled to a node receive an IP address from the node's Pod CIDR range. At this point pods can not communicate with other pods running on different nodes due to missing network [routes](https://cloud.google.com/compute/docs/vpc/routes).

In this lab Flannel is chosen to add overlay network interface. 

Run following command on one the controller node:

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

it would take up to few minutes to the flannel pods to go up. We could verify by querying the pod:

```
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                          READY   STATUS              RESTARTS   AGE
kube-system   kube-flannel-ds-amd64-7b4h6   1/1     Running             0          110s
kube-system   kube-flannel-ds-amd64-bqw7v   1/1     Running             0          110s
kube-system   kube-flannel-ds-amd64-jp2jn   1/1     Running             0          110s
```

Next: [Deploying the DNS Cluster Add-on](12-dns-addon.md)
