---
title: "Proxmox Notes"
date: 2023-12-19
tags:
  - proxmox
---

## Networking

### NAT config

For my setup, I have one physical Ethernet connection on `eno1` device, which by default is assigned to `vmbr0`. 

I configured a NAT bridge `vmbr1` by following the instructions from the [Proxmox Network config Guide](https://pve.proxmox.com/wiki/Network_Configuration#sysadmin_network_masquerading).

The config for `vmbr1` is as follows:
```bash
auto vmbr1
iface vmbr1 inet static
        address 203.0.113.0/24
        bridge-ports none
        bridge-stp off
        bridge-fd 0
        post-up   echo 1 > /proc/sys/net/ipv4/ip_forward
        post-up   iptables -t nat -A POSTROUTING -s '203.0.113.0/24' -o vmbr0 -j MASQUERADE
        post-down iptables -t nat -D POSTROUTING -s '203.0.113.0/24' -o vmbr0 -j MASQUERADE
```

After config, restart the networking service:
```bash
systemctl restart networking
```

### VLAN config

I wanted to set up VLAN for testing purposes and found this useful thread on the [Proxmox Forum](https://forum.proxmox.com/threads/vlan-configuration-in-cluster-pve-8-0-3.129543/). 

The config is similar to the NAT setup but includes VLAN-specific settings:

```bash
 auto vmbr1
 iface vmbr1 inet static
         address 203.0.113.0/24
         bridge-ports none
         bridge-stp off
         bridge-fd 0

         post-up   echo 1 > /proc/sys/net/ipv4/ip_forward
-        post-up   iptables -t nat -A POSTROUTING -s '203.0.113.0/24' -o vmbr0 -j MASQUERADE
+        post-up   iptables -t nat -A POSTROUTING -s '203.0.0.0/16' -o vmbr0 -j MASQUERADE
-        post-down iptables -t nat -D POSTROUTING -s '203.0.113.0/24' -o vmbr0 -j MASQUERADE
+        post-down iptables -t nat -D POSTROUTING -s '203.0.0.0/16' -o vmbr0 -j MASQUERADE

+auto vmbr1.114
+iface vmbr1.114 inet static
+    address 203.0.114.1
+    netmask 255.255.255.0
```

## Managing ISO and VM Images

### Uploading ISO

To upload an ISO file to Proxmox, follow the instructions detailed in the [OVHcloud Support](https://support.us.ovhcloud.com/hc/en-us/articles/360010916620-How-to-Create-a-VM-in-Proxmox-VE#upload): Go to `local` > `Upload` > `Select File`.

### Importing ovf

For setting up GNS3 on Proxmox, I downloaded the GNS3 VM from [GNS3 VM Download](https://www.gns3.com/software/download-vm). 

Since GNS3 VM is available only in VMware ESXi (OVF) format, use the following command to import it to Proxmox, as explained in Zach Grace's [Proxmox Cheat Sheet](https://zachgrace.com/cheat_sheets/proxmox/):

```bash
qm importovf <vmid> <ovf file> <storage>
```

### Enabling Nested Virtualization

Nested virtualization can be enabled in Proxmox by referring to the [Proxmox Nested Virtualization Guide](https://pve.proxmox.com/wiki/Nested_Virtualization#Enable_Nested_Hardware-assisted_Virtualization):

```bash
qm set <vmid> --cpu host
```

### Creating VM with Cloud Image and Cloud-Init

To create a VM using a cloud image and cloud-init, use the steps provided in the [Proxmox Cloud-Init Support](https://pve.proxmox.com/wiki/Cloud-Init_Support):

Create as Template
```bash
wget https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64.img
qm create 9000 --memory 2048 --net0 virtio,bridge=vmbr0 --scsihw virtio-scsi-pci
qm set 9000 --scsi0 local-lvm:0,import-from=/path/to/bionic-server-cloudimg-amd64.img
qm set 9000 --ide2 local-lvm:cloudinit
qm set 9000 --boot order=scsi0
qm set 9000 --serial0 socket --vga serial0
qm template 9000
```

Create VM
```bash
qm clone 9000 123 --name ubuntu2
qm set 123 --sshkey ~/.ssh/id_rsa.pub
qm set 123 --ipconfig0 ip=10.0.10.123/24,gw=10.0.10.1
```

## Updating Proxmox Without Subscription

The default repository requires a paid subscription for Proxmox updates. 

However, it's possible to bypass this by manually changing the repository settings, as outlined in the [VirtualizationHowTo Guide](https://www.virtualizationhowto.com/2022/08/proxmox-update-no-subscription-repository-configuration/).

vim /etc/apt/sources.list.d/pve-enterprise.list

```
-deb https://enterprise.proxmox.com/debian/pve bookworm enterprise
+deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription
```

vim /etc/apt/sources.list.d/ceph.list
```
-deb https://enterprise.proxmox.com/debian/ceph-quincy bookworm enterprise
+deb http://download.proxmox.com/debian/ceph-quincy bookworm no-subscription
```
