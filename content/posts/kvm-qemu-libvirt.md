---
title: "KVM - QEMU, libvirt"
date: 2023-12-04
tags:
- kvm
- libvirt
- qemu
---

Purpose of this note: The existing online tutorials provide limited information about KVM.

Many tutorials reference the use of virt-manager, which is akin to a GUI tool for creating virtual machines.

However, my interest lies in a deeper understanding of KVM, which these tutorials do not adequately cover.

I would like to begin with 
- a high-level overview
- break down KVM into different aspects and then 
- combine all the details into a lab

## Setup Specification

Regarding the upcoming commands/guide, I am using Ubuntu 22.04 as my host operating system, and the guest OS is openSUSE 15.6.

## Objectives

- Focus primarily on QEMU, though insights into libvirt are also valuable, as its underlying configurations can aid in verifying settings.
- Gain an understanding of the differences and uses of serial vs. monitor in QEMU, as well as the console.
- Learn about VNC and SPICE in QEMU.
- Comprehend networking aspects within KVM.

## Methodology

Creating VMs with KVM can be approached in two ways:

1. Libvirt - Higher Level:
   - Utilizes the libvirtd daemon, allowing VM and network definitions (in XML) to persist and enabling autostart on boot.
   - The default bridge network `default` comes with bridge device named `virbr0`.

2. QEMU - Lower Level:
   - Operates solely through the command line.
   - For networking, it involves using a helper tool to add a tap network to a bridge.

## Resources for reference

I've found two GitHub projects that have been tremendously helpful in my study from scratch. The labs I've worked on are essentially based on these projects:

1. [create-vm](https://github.com/earlruby/create-vm) - Using libvirt
2. [vm-provision](https://github.com/racingmars/vm-provision) - Leveraging QEMU

In addition to these projects, I've also come across three command builders for QEMU and virt-install that, despite not being updated for a while, are incredibly valuable for studying:

- [Command Line Libvirt](https://www.nongnu.org/pretest/command-line-libvirt.html)
- [Command Line QEMU](https://www.nongnu.org/pretest/command-line-qemu.html)
- [QEMU Command Generator](https://qemu-command-generator.netlify.app/)

For in-depth information and documentation, you can refer to the following:

- The [man page of qemu-system-x86_64](https://www.qemu.org/docs/master/system/invocation.html#sec-005finvocation)
- The [wiki page of QEMU](https://wiki.gentoo.org/wiki/QEMU/Options#Processor) provides a wealth of details and examples.
- On the [Arch Linux wiki](https://wiki.archlinux.org/title/QEMU), you'll find a plethora of resources.

Additionally, if you're looking for guidance on using `virt-install`, RedHat has a comprehensive guide available:

- [RedHat's Virt-Install Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/sect-guest_virtual_machine_installation_overview-creating_guests_with_virt_install#sect-Guest_virtual_machine_installation_from_ISO_image)

For installation guides, you can follow these resources:

- Ubuntu: [Virtualization with Libvirt](https://ubuntu.com/server/docs/virtualization-libvirt)
- CentOS: [Creating Guest VMs on Linux KVM](https://www.thegeekstuff.com/2014/10/linux-kvm-create-guest-vm/)

## Prepartion

Preparation for create VM with KVM:

- Install related software packages
- Cloud image
- Disk image
- Cloud-init ISO

First, ensure qemu and virt-install are installed. 

Then, obtain a cloud image to serve as the base disk. 

Create a copy of this image (backed by the cloud image) to form the disk image for our VM. 

Configure the user and network settings using cloud-init, which is packed into an ISO and attached to the VM as a CD-ROM.

## QEMU Shortcuts

In QEMU, there are many ways to achieve the same thing with different parameters and numerous shortcuts. This can be quite confusing as different tutorials, websites, and discussions often use varying methods.

For clarity, I prefer to convert the parameters to their shortcut forms:

```bash
-hda = -drive
-cdrom = -drive
-drive = -blockdev -device
```

```bash
-audio = -audiodev -device
-nic = -netdev -device
```

For example:

```bash
-drive file=linux.img,format=qcow2,if=virtio
```

is equivalent to:

```bash
-hda linux.img
```

And:

```bash
-drive file=cloud-init.img,media=cdrom,if=virtio
```

is the same as:

```bash
-hda cloud-init.img
```

Also:

```bash
-chardev socket,id=char0,path=u1-serial,server=on,wait=off
-serial chardev:char0
```

can be shortened to:

```bash
-serial unix:u1-serial,server,nowait
```

Furthermore:

```bash
-netdev bridge,id=hn0 
-device virtio-net-pci,netdev=hn0,id=nic1
```

```bash
-netdev tap,helper=/usr/local/libexec/qemu-bridge-helper,id=hn0 
-device virtio-net-pci,netdev=hn0,id=nic1
```

are equivalent to:

```bash
-nic bridge,model=virtio-net-pci
```

## Networking

The common network types for VM access include:
- User
- Bridge/Tap

From my perspective, tap and bridge networks function similarly. When using a bridge, the qemu bridge-helper script creates a tap device and adds it to the bridge. In the case of a tap, up and down scripts achieve a similar outcome.

The user network type is akin to NAT. To SSH into the VM, setting up port forwarding is necessary.

To create a network using libvirt, configure it with a DHCP server using dnsmasq. The following commands are used:

Creates the network temporarily

```bash
virsh create <network-config>
```

Creates the network permanently and ensures the network starts automatically

```bash
virsh define <network-config>
virsh net-autostart <network>
```

Starts the defined network

```bash
virsh start <network>
```

However, with qemu alone, no network is created automatically. The user must manually set up everything, 

To set up a network on QEMU, you need to:

1. Create the network device.
2. Set up a DHCP server (if needed), typically by creating a bridge.

There are various methods for this:
- Temporary setup using iproute2, which is easy but not persistent.
- Persistent setups differ across operating systems and are not well documented. For instance, 
  - Ubuntu uses netplan, 
  - older versions or Debian use `/etc/network/interfaces`, and 
  - CentOS/RedHat uses `/etc/sysconfig/network/ifcfg-<dev>`
- A hack way is using libvirt for network creation.

I've gathered some useful configurations from the web:

Ubuntu (netplan)
- [bunch of example](https://github.com/canonical/netplan/tree/main)

Centos / Redhat (`/etc/sysconfig/network/`)
- [basic](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide/sec-configuring_ip_networking_with_ifcg_files)
- [bridge](https://cmdref.net/middleware/virtualization/kvm/network_brx_basic.html#configuration)
- [bond + bridge + vlan](https://cmdref.net/middleware/virtualization/kvm/network_brx_vlan.html#configuration)

Opensuse - wicked (`/etc/sysconfig/network/`) - I can't found a web page related to this
- [bunch of examples](https://github.com/openSUSE/wicked/blob/master/man/ifcfg-ovs-bridge.5.in)

Debian / Oldder Ubuntu (`/etc/network/interfaces`)
- [basic](https://www.cyberciti.biz/faq/setting-up-an-network-interfaces-file/)
- [bridge](https://wiki.debian.org/NetworkConfiguration#Bridging)
- [vlan](https://wiki.debian.org/NetworkConfiguration#Network_init_script_config)
- [bond](https://wiki.debian.org/NetworkConfiguration#bonding_with_active_backup)

For DHCP, if an existing one is available, that's good, but if you need one for testing, you can use libvirt to create a bridge network and DHCP server.

For those not using libvirt, here's a snippet to create a temporary DHCP server with 

```bash
dnsmasq --interface=br0 --bind-interfaces --dhcp-range=172.20.0.2,172.20.255.254
```

Ref: [QEMU Host-only Networking guide on the Arch Linux wiki](https://wiki.archlinux.org/title/QEMU#Host-only_networking).

## Cloud Init

Since the cloud image doesn't have a user password configuration, logging into the system via console isn't possible. 

Therefore, SSH is necessary to access the system. 

This requires the VM to have network connectivity and the user to have the SSH key or password set on the system. This setup can be achieved using cloud-init.

To facilitate this, at least three files are required:
- `meta-data`: [meta-data](https://cloudinit.readthedocs.io/en/latest/tutorial/qemu.html#define-our-metadata)
- `user-data`: [user-data](https://cloudinit.readthedocs.io/en/latest/tutorial/qemu.html#define-our-user-data)
- `network-config`: [network-config](https://cloudinit.readthedocs.io/en/latest/reference/network-config-format-v1.html)

These files can either be packed into an ISO and attached to the VM, or served to the VM through a simple HTTP server.

I note that for openSUSE, cloud-init's network-config version 2 doesn't work, so version 1 must be used.

## Display - VNC and Spice

Virt-viewer can be utilized for both VNC and Spice connections.

Ensure that X11 forwarding is enabled on ssh server in the `/etc/ssh/sshd_config` file.
On client, this can be activated with

```bash
ssh -X
```

Since I'm using macOS, I have installed XQuartz as X11 Client.

For the lab purposes, I'm using the root account, so I use

```bash
sudo -E bash
```
to maintain the $DISPLAY variables for X11.

It's crucial to set XQuartz's X11 preference settings correctly. 

Specifically, in the output settings > colors, select 'from display' and avoid options like 256color.

Previously, I had set it to 256color to speed up the x2go redning, but this caused issues when connecting to Spice (the screen is blank).

Most of the commands for Spice are mentioned in the official user manual, which can be found at [Spice User Manual](https://www.spice-space.org/spice-user-manual.html).

For instance, to start Spice, use the command:

```bash
-vga qxl
-spice port=3001
-soundhw hda
-device virtio-serial
-chardev spicevmc,id=vdagent,debug=0,name=vdagent
-device virtserialport,chardev=vdagent,name=com.redhat.spice.0
```

Then, connect to the Spice server using:

```bash
remote-viewer spice://127.0.0.1:3001
```

Other features like password authentication, TLS, and SASL authentication add additional security to the connection, which is good to know.

In the future, I would also like to explore USB and audio redirection/pasteboard functionalities.

## Serial and Console

- Serial refers to the VM console, which I frequently use.
- Console refers to the QEMU console, which I rarely use for myself.

There are different ways to connect to serial/console, and I'll list those I have experience with:

- socket
- telnet
- pty
- stdio

Commands for connecting to serial/console:

```bash
-serial unix:u1-serial,server,nowait
-serial stdio,server,nowait
-serial pty,server,nowait
-serial telnet::7000,server,nowait
```

To connect to serial from socket:

```bash
socat - UNIX-CONNECT:u1-serial
```

To connect to serial from telnet:

```bash
telnet host port
```

To connect to serial from pty:

```bash
screen /dev/pts/2
```

However, this can be messy for me. The style is often broken as it doesn't resolve colors and newlines correctly.

Setting `export TERM=screen-256color` improves the color issue but doesn't resolve the newline problem.

stdio:

- Useful for running QEMU in the foreground (without `--daemonize`) for quick testing.

## Labs

### Preparation

Software Packages:

For QEMU:

```bash
apt install -y qemu-kvm
```

For libvirt:

```bash
apt install -y qemu-kvm libvirt-daemon-system virtinst
```

To create cloud-init ISO:

```bash
apt install -y genisoimage
```

Cloud Image:

```bash
wget https://download.opensuse.org/repositories/Cloud:/Images:/Leap_15.5/images/openSUSE-Leap-15.5.x86_64-1.0.0-NoCloud-Build1.143.qcow2
```

Disk Image:

```bash
qemu-img create -b openSUSE-Leap-15.5.x86_64-1.0.0-NoCloud-Build1.143.qcow2 -f qcow2 -F qcow2 "linux.img" "20G"
```

Cloud-init ISO:

```bash
mkdir /lab/cloud-init -p
```

```bash
cat << EOF | tee /lab/cloud-init/network-config
network:
  version: 1
  config:
  - type: physical
    name: eth0
    subnets:
      - type: dhcp
  - type: physical
    name: eth1
    subnets:
      - type: dhcp
EOF
```

```bash
cat << EOF | tee /lab/cloud-init/user-data
#cloud-config
users:
  - name: <user>
    sudo: ["ALL=(ALL) NOPASSWD:ALL"]
    groups: sudo
    shell: /bin/bash
    homedir: /home/hugotse
    ssh_authorized_keys:
      - "<SSH_PUBLIC_KEY>"
EOF
```

```bash
cat << EOF | tee /lab/cloud-init/meta-data
instance-id: u1
local-hostname: u1
EOF
```

```bash
genisoimage -output "/lab/cloud-init.img" -volid cidata -rational-rock -joliet /lab/cloud-init/*
```

### Libvirt

To create VM with libvirt:

Create a bridge network:

```bash
cat << EOF | tee /lab/bridge.xml
<network connections='2'>
  <name>br0</name>
  <forward mode='nat'/>
  <bridge name='br0' stp='on' delay='0'/>
  <mac address='52:54:00:8a:e9:7c'/>
  <ip address='192.168.123.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.123.2' end='192.168.123.254'/>
    </dhcp>
  </ip>
</network>
EOF
```

```bash
chmod u+s /usr/lib/qemu/qemu-bridge-helper
```

```bash
mkdir /etc/qemu/
```

```
cat << EOF | tee /etc/qemu/bridge.conf
allow br0
EFO
```

```bash
virsh net-define bridge.xml
virsh net-start br0
virsh net-autostart br0
```

```bash
virt-install \
--name="u1" \
--import \
--disk "path=linux.img,format=qcow2" \
--disk "path=cloud-init.img,device=cdrom" \
--ram="2048" \
--vcpus="2" \
--osinfo opensuse15.0 \
--wait 0 \
--noautoconsole \
--autostart \
--network bridge=br0
```

Connect to console
```bash
virsh console u1
```

Connect to the display

```bash
virt-viewer -u u1
```

Stop & Stop the VM

```bash
virsh start u1
virsh shutdown u1
```

Remove the VM

```bash
virsh destroy u1
virsh undefine u1
```

### QEMU

To create a VM with QEMU, combine everything together:

```bash
qemu-system-x86_64 \
-name u1 \
# Enable KVM
-enable-kvm \
# Add disk image
-hda linux.img \
# Add cloud init
-cdrom cloud-init.img \
# Config Memory and CPU
-m 2048 \
-smp 2 \
# First network - user network with port forward SSH to TCP 1025
-nic user,hostfwd=tcp:127.0.0.1:1025-:22 \
# Second network - bridge network on br0 device
-nic bridge,br=br0,model=virtio-net-pci \
# Spice
-vga qxl \
-spice port=3001,password=123 \
-soundhw hda \
-device virtio-serial \
-chardev spicevmc,id=vdagent,debug=0,name=vdagent \
-device virtserialport,chardev=vdagent,name=com.redhat.spice.0 \
# Serial
-serial stdio
```
