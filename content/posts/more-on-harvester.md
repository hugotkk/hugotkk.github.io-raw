---
title: "More on harvester"
date: 2023-12-16
tags:
- harvester
- longhorn
- rancher
---

I further explored the Harvester system and made some updates based on my discoveries.

## Storage

### Expanding the Disk
Initially, we had one disk called `/dev/sda`, and Longhorn storage used the last partition, which was `/dev/sda6`.

To expand the volume, I followed a guide called [AWS EC2 User Guide on Recognizing Expanded Volume in Linux](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/recognize-expanded-volume-linux.html#extend-file-system). This process involves two main steps:

1. Expanding the partition.
2. Resizing the file system.

However, since the `growpart` utility was not available, I used `parted` instead. Here are the steps I took:

- I deleted the 6th partition (`/dev/sda6`).
- Then, I recreated the partition.
- Finally, I saved the changes.

### Adding Additional Disks into Harvester

I followed the instructions in the [Harvester Documentation](https://docs.harvesterhci.io/v1.2/host/) to add another disk to Harvester. But after adding the new disk (`/dev/sdb`), it didn't show up in the web interface.

I fixed this problem by setting the `auto-disk-provision-paths` to `/dev/sdb` in the settings, as explained in the [Harvester Advanced Documentation](https://docs.harvesterhci.io/v1.2/advanced/index#auto-disk-provision-paths-experimental). 

I'm going to look into this issue further to understand it better.

### Adjusting Replication in Longhorn Storage

To change the replication settings in Longhorn storage, I did the following:

1. created a new storage class and set the number of replicas to 1.
2. used this new storage class when creating images and volumes.

After making these changes, I could see in the Longhorn Dashboard that there was only one replica.

It's worth noting that when setting up a PV on RKE2, the system usually uses the default storage class. To make things consistent, I made this new storage class the default one.

### Enable Longhorn UI Dashboard

To enable the Longhorn UI Dashboard:

1. Initially, the Longhorn UI dashboard was not enabled by default. To turn it on, follow these steps:
   - Start by enabling the support bundle as explained in the documentation.
   - Then, you can access the UI from the "support" section.

For detailed guidance, please consult the [Harvester Documentation on Generating a Support Bundle](https://docs.harvesterhci.io/v1.2/troubleshooting/harvester#generate-a-support-bundle).

### Optimize reserved CPU, memory, and storage:

You can make adjustments to the reserved CPU, memory, and storage settings within the Longhorn UI.

## High Availability

### HA in VMs & Volumes

**Testing Live Migration:**
- During live migration, the VM moves to a different host while the storage remains on the original host.

**Scenario with Single Replica Volume:**
- I created a VM with a volume having only one replica.
- Initially, both the storage and VM were on the same node.
- Then, I disabled the storage engine (disable scheduling and started eviction via Longhorn UI), causing the storage to relocate to a different host while the VM continued to operate normally.

This demonstrates the independence of VM and storage HA and their ability to function on separate nodes.

**Impact of Storage Node Shutdown:**

- When I shut down the host containing the sole storage, the VM became inoperative.

This indicates that if all available storage becomes inaccessible, the VM will stop functioning.

**Testing with Multiple Replicas:**

- In cases where the volume has multiple replicas (e.g., 3) and the host with the storage is powered down, the VM remains operational. A new replica is created to maintain the required number of replicas.

This showcases the HA feature within the volume.

**VM Host Shutdown Scenario:**

- Shutting down the host where the VM resides changes the VM status to "Not Ready." Subsequently, the VM is relocated to another node and returns to normal operation.

This scenario illustrates the HA capabilities of VMs.

### HA in Host Management

**Master Node Promotion Limitations:**
- I noticed that there is no direct method to manually promote a master node.
- Master node promotion only occurs when an existing master node is removed from Harvester.
- Additionally, there is no provision for adding extra master nodes to the system.
- This configuration restricts my control over which nodes can be promoted or excluded from promotion.

## Rancher Integration

**Deploying RKE2 with Rancher:**
- I have successfully deployed RKE2 using Rancher.
- Cluster scaling operations function smoothly.
- Upgrading RKE2 has been successful without major issues.
- Testing backup and restoration processes for the etcd database yielded positive results.
- The integration of PV in an RKE2 cluster (downstream) with Longhorn storage within Harvester (upstream) has been successful.

Although there have been occasional bugs, the overall process has been satisfactory.

## Networking

### Setup vlan

**Harvester config:**
- Configuring VLAN on Harvester is relatively straightforward.
- It involves creating a new VM network and assigning a VLAN ID.

**Proxmox config (for Hosting Harvester):**
- To establish VLANs on the bridge interface within Proxmox, additional settings are necessary.
- I utilized `vmbr1` as the bridge interface for all Harvester hosts.

For setting up NAT on Proxmox, you can refer to the guide provided at this link: [Network config - sysadmin network masquerading](https://pve.proxmox.com/wiki/Network_Configuration#sysadmin_network_masquerading).

For discussions related to VLAN config on the Proxmox bridge, you can find valuable insights and assistance in this forum thread: [VLAN config in Cluster PVE 8.0.3](https://forum.proxmox.com/threads/vlan-configuration-in-cluster-pve-8-0-3.129543/).


vi /etc/network/interfaces
```
 auto vmbr1
 #private sub network
 iface vmbr1 inet static
         address  203.0.113.1/24
         bridge-ports none
         bridge-stp off
+        bridge-vlan-aware yes
+        bridge-vids 114
         bridge-fd 0
 
         post-up   echo 1 > /proc/sys/net/ipv4/ip_forward
-        post-up   iptables -t nat -A POSTROUTING -s '203.0.0.0/24' -o vmbr0 -j MASQUERADE
+        post-up   iptables -t nat -A POSTROUTING -s '203.0.0.0/16' -o vmbr0 -j MASQUERADE
         post-up   iptables -t raw -I PREROUTING -i fwbr+ -j CT --zone 1
-        post-down iptables -t nat -D POSTROUTING -s '203.0.0.0/24' -o vmbr0 -j MASQUERADE
+        post-down iptables -t nat -D POSTROUTING -s '203.0.0.0/16' -o vmbr0 -j MASQUERADE
         post-down iptables -t raw -D PREROUTING -i fwbr+ -j CT --zone 1

+auto vmbr1.114
+iface vmbr1.114 inet static
+    address 203.0.114.1/24
```

**DHCP Server config (Ubuntu):**
- Add VLAN settings to the Netplan config in `/etc/netplan/50-cloud-init.yaml`.
- Define VLAN `114` on interface `ens19` with an IP range of `203.0.114.182/24`.

```
+ vlans:
+     ens19.114:
+         id: 114
+         link: ens19
+         addresses: [203.0.114.182/24]
```

**Dnsmasq config:**
```
-interface=ens19,lo
+interface=ens19,ens19.114,lo
...
+dhcp-range=ens19.114,203.0.114.2,203.0.114.254
+dhcp-option=ens19.114,3,203.0.114.1
+dhcp-host=BC:24:11:9B:39:4A,test,203.0.114.185
```

### Setup bond

**Post-Installation Network Bonding:**
- I experimented with configuring network bonding on Harvester after its installation.
- For detailed guidance, I referred to the [Harvester Documentation on Updating Configuration](https://docs.harvesterhci.io/v1.2/install/update-harvester-configuration#configuration-persistence-3).

**Bonding Setup During Installation:**
- The Harvester documentation also provides instructions for setting up bonding during the installation process. This can be found in the [Harvester Configuration Section](https://docs.harvesterhci.io/v1.2/install/harvester-configuration#installmanagement_interface).

````
 install:
   mode: create
   management_interface:
     interfaces:
     - name: ens18
+    - name: ens10    
     method: dhcp
     bond_options:
       mode: active-backup
       miimon: 100
```

- In this setup, the bond config utilizes the MAC address of the first listed interface (in this case, `ens18`).
- Consequently, the DHCP config should align with the MAC address of the first interface to ensure proper functioning.