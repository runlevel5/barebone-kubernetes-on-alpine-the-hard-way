# Prerequisites

## Alpine Linux

It is required that you have access to computers or VMs running Alpine (edge). It is highly recommended you setup VMs running under VirtualBox or KVM/QEMU.

At the time of the writing, Alpine edge has all the up-to-date kubernetes packages. Please enable Alpine edge repositories:

```sh
echo "http://dl-cdn.alpinelinux.org/alpine/edge/main" > /etc/apk/repositories
echo "http://dl-cdn.alpinelinux.org/alpine/edge/community" >> /etc/apk/repositories
echo "http://dl-cdn.alpinelinux.org/alpine/edge/testing" >> /etc/apk/repositories
```

Next: [Installing the Client Tools](02-client-tools.md)
