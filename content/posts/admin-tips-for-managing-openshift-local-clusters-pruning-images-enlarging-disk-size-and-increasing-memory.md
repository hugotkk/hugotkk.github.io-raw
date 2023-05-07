---
title: "Admin Tips for Managing OpenShift Local Cluster: Pruning Images, Enlarging Disk Size, and Increasing Memory"
date: 2023-05-07
tags:
- k8s
- openshift-local
---

## Real Hardware Requirments

Suggestion from [doc](https://access.redhat.com/documentation/en-us/red_hat_openshift_local/2.12/html/getting_started_guide/installation_gsg#doc-wrapper):

- 4 physical CPU cores
- 9 GB of free memory
- 35 GB of storage space

Recommended (to make it work for development):

- at least 15GB memory. 18GB is better.
- 60 GB storage

## Increase Memory:

```
crc stop
crc config set memory <memory_in_mb>
crc start
```

## Prune images in the cluster:

- OpenShift Local uses Podman to run containers for the infrastructure of the cluster.
- Image cache accumulates and grows quickly.
- Need to SSH into the KVM and prune the images.

Access the host:

- Use the SSH command and the key (`~/.crc/machines/crc/id_ecdsa`) found in the .crc folder: ([See](https://docs.openshift.com/container-platform/4.8/networking/accessing-hosts.html))

```
ssh -i <key> core@<master-hostname>
```

To find the master ip:

- Create a pod with hostNetwork
- install `iproute2`
- use `ip addr` to find the host IP.

```
apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  hostNetwork: true
  containers:
  - name: alpine
    image: alpine
    tty: true
```

Prune dangling images:

```
podman image prune
```

## Enlarge disk size: (Thanks to this [discussion](https://github.com/crc-org/crc/issues/127))

1. Enlarge the KVM image:

```
CRC_MACHINE_IMAGE="$HOME/.crc/machine/crc/crc"
crc stop
qemu-img resize ${CRC_MACHINE_IMAGE} +24G
```

2. Find the root partition (the largest one) using `virt-filesystems`:

```
virt-filesystems --long -h --all -a $HOME/.crc/machines/crc/crc
Name       Type        VFS      Label  MBR  Size  Parent
/dev/sda1  filesystem  unknown  -      -    1.0M  -
/dev/sda2  filesystem  ext4     boot   -    1.0G  -
/dev/sda3  filesystem  xfs      root   -    39G   -
/dev/sda1  partition   -        -      -    1.0M  /dev/sda
/dev/sda2  partition   -        -      -    1.0G  /dev/sda
/dev/sda3  partition   -        -      -    39G   /dev/sda
/dev/sda   device      -        -      -    40G   -
```

3. Enlarge the partition in KVM:

```
cp ${CRC_MACHINE_IMAGE} ${CRC_MACHINE_IMAGE}.ORIGINAL
virt-resize --expand /dev/sda3 ${CRC_MACHINE_IMAGE}.ORIGINAL ${CRC_MACHINE_IMAGE}
```




