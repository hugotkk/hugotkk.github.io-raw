---
title: "Import qcow2 cloud image to ESXi"
date: 2023-12-19
tags:
  - kvm
  - ESXi
---

I'm attempting to configure CentOS Stream 9 on ESXi using a cloud image. 

However, the cloud image is not available in VMDK format, and the ISO file is quite large at 9GB. 

I'd prefer not to install the VM from the ISO.

Instead, I plan to convert the QCOW2 image to VMDK format and then import it into ESXi. To begin, I need to download the QCOW2 cloud image: https://cloud.centos.org/centos/9-stream/x86_64/images/

I followed the steps outlined in this guide ([https://blog.ktz.me/migrate-qcow2-images-from-kvm-to-vmware/](https://blog.ktz.me/migrate-qcow2-images-from-kvm-to-vmware/)) to convert a VMDK from a QCOW2 image:

1. Use the `qemu-img` tool to convert the QCOW2 image to a VMDK format:
```bash
qemu-img convert -f qcow2 -O vmdk quassel.qcow2 quasselog.vmdk
```

2. Copy the resulting VMDK file to ESXi.

3. In ESXi, create a VM with no disk attached.

4. Perform an additional conversion of the VMDK file to a ESXi-compatible format using `vmkfstools`:
```bash
vmkfstools -i quasselog.vmdk -d thin quassel.vmdk
```

5. After this conversion, I got two files. Copy both of them to the VM's datastore.

6. Edit the VM's settings, add a new disk, and select the option to use an existing disk. Choose the converted VMDK file to attach it to the VM.

The cloud image does not automatically assign a default linux account / password, leaving us with no means to access the system.

There are two methods available for logging into the VM:

1. Use cloud-init to configure the network and user
2. Set a default root password within the QCOW2 image

**Method1**:

Start by creating a `user-data` file:

vim user-data

```yaml
#cloud-config
users:
  - name: centos
    plain_text_passwd: centos
    groups: sudo
    sudo: ALL=(ALL) NOPASSWD:ALL
    shell: /bin/bash
    lock_passwd: false
```

See more on [cloud-init User-Data Examples](https://cloudinit.readthedocs.io/en/latest/reference/examples.html).

Next, convert the `user-data` file to base64 encoding:
```
base64 user-data
```

In ESXi, modify the VM's settings:
- Navigate to Configuration Parameters.
- Add the following parameters:

```
guestinfo.userdata.encoding: base64
guestinfo.userdata: <base64 encoded user-data>
```

Include the network configuration
```yaml
network:
  version: 2
  ethernets:
    eth1:
      dhcp4: false
      dhcp6: false
      addresses:
        - 192.168.0.100/24
      gateway4: 192.168.0.100
      nameservers:
        addresses:
          - 8.8.8.8
```

See more on [cloud-init Network Config Format v2 Examples](https://cloudinit.readthedocs.io/en/latest/reference/network-config-format-v2.html#examples).

Finally, add this data to the `guestinfo.metadata` to the Configuration Parameters.

See more on [cloud-init VMware Data Source](https://cloudinit.readthedocs.io/en/latest/reference/datasources/vmware.html#guestinfo-keys).

**Method2**:

```bash
virt-customize -a centos.qcow2 --root-password password:<password>
```

See more on [Use libguestfs to manage virtual machine disk images](https://www.redhat.com/sysadmin/libguestfs-manage-vm)