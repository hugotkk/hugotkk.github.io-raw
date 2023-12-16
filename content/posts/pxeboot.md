---
title: "pxeboot"
date: 2023-12-16
tags:
- pxeboot
---

I recently experimented with PXE boot and auto-installation for various OS in my lab setup. Here is the process and details sharing with you.

## Netboot install process

- When the VM is started, it enters the UEFI.
- The VM sends a DHCP request to the broadcast address.
- The DHCP server responds to this request by leasing an IP address to the VM. Along with the IP address, the DHCP server provides the location of the boot image (`grubx64.efi` for UEFI boot).
- The VM then downloads the boot image from the PXE server and start the GRUB2 bootloader.
- GRUB2 loads the `grub.cfg` file, which contains the config for the boot menu.
- The boot menu appears, presenting the user with different OS options to install.
- Once the user selects the desired OS, the installation process begins.

## Workflow

1. Configure the dnsmasq, tftp, and Apache servers.
2. Set up DHCP for a test VM.
4. Prepare the boot loader.
3. Customize the test VM to boot using grub2 and load the grub2.cfg file, optionally including a dummy menu entry.
5. Prepare the ISO, extract the kernel, initrd, repo and retrieve the boot command from the official documentation.
6. If feasible, conduct a manual installation using qemu with a CD-ROM to obtain the auto-installation script.
7. Further develop the auto-installation script within the qemu environment.
8. Update the auto-installation script
9. Update the grub2.cfg file - crafting the appropriate menu entry based on the qemu command.
10. Commence the netboot installation in a real environment.

## Lessions Learnd

### Separating netboot and auto install in testing

When conducting tests, it's advisable to separate the PXE boot and auto-installation processes.

The primary objective of PXE booting is to ensure that the VM acquires the correct IP address and that the boot loader loads correctly.

To effectively test the auto-installation, it's best to do so independently.

You can carry out the auto-installation testing using QEMU/KVM.

By directly initiating the VM with specific kernel, initrd, and boot options, you can bypass the PXE boot and boot loader stages, proceeding directly into the installation process.

This approach not only saves time by eliminating the need to wait for the BIOS and boot loader but also isolates any potential network issues related to PXE booting.

### Debugging with Serial Display

Initially, I opted to connect a serial port to the VM for debugging purposes.

However, it's important to note that many installers and kernels have limited support for serial output, often resulting in a blank screen.

As a result, I transitioned to using X11 forwarding for GUI display on my local PC and stopped relying on the serial port unless it was explicitly recommended.

### Using CD-ROM as a Troubleshooting Fallback

In cases where issues arise with QEMU, attaching a CD-ROM to the VM can help identify potential misconfigurations. CD-ROM installation should be considered as a last resort.

For instance, I encountered difficulties when attempting to initiate XCP-ng installation with UEFI, even when using a CD-ROM. This may indicate possible hardware compatibility issues.

### Enabling Logging

Enabling logs for DHCP, TFTP, and Apache is a crucial step for troubleshooting.

These logs provide in-depth insights into the booting and installation processes.

When examining the TFTP logs, you can monitor requests for the boot loader (`grubx64.efi`), boot menu (`grub.cfg`), and kernel/ramdisk, aiding in the identification of required config and pinpointing any misconfigurations.

Simultaneously, the Apache server's access logs prove invaluable for comprehending the installation process. 

You can observe the progress of the installation, such as the loading of the ISO or the retrieval of packages from the server.

This level of visibility is essential for ensuring that the installation proceeds as anticipated and for diagnosing any potential irregularities.

### Obtain the auto-install script from manual installation (if feasible)

For instance, in the case of Ubuntu:

When installation ubuntu using server installer, an auto-install file that allows you to replicate the installation process is generated and can be found at `/var/log/installer/autoinstall-user-data`.

https://ubuntu.com/server/docs/install/autoinstall

For CentOS:

After the installation is successfully completed, all the selections made during the installation process are saved in `/root/anaconda-ks.cfg`.

https://docs.centos.org/en-US/centos/install-guide/Kickstart2/#sect-kickstart-file-create

### Universal Boot loader

There are multiple ways to obtain the boot loader image. You can either install the required packages or copy them from an existing server. 

However, it's important to note that different Linux distributions may compile these components differently, potentially missing necessary features and using varying config paths.

As an illustration, let's consider how Ubuntu and CentOS handle their Grub configurations:

In the case of Ubuntu, Grub is loaded from a subdirectory named "grub" within the root directory ("/grub/grub.cfg").

https://ubuntu.com/server/docs/install/netboot-amd64

> Create /srv/tftp/grub/grub.cfg that contains

Meanwhile, CentOS loads Grub directly from the root directory ("/grub.cfg").

https://docs.centos.org/en-US/centos/install-guide/pxe-server/#sect-network-boot-setup-uefi

> Add a configuration file named grub.cfg to the tftpboot/ directory. A sample configuration file at /var/lib/tftpboot/grub.cfg might look like:

Following the official guides from each distribution can sometimes be confusing and may not work seamlessly when dealing with netboot across different distributions using a single bootloader.

One valuable feature I've encountered in the legacy boot loader `pxelinux.0` is its ability to load config files based on MAC addresses. It automatically searches for the MAC address and, if not found, gracefully defaults to `default`. This functionality proves exceptionally useful when booting various machines with distinct menu configs.

More information on this feature can be found here: [PXELINUX config](https://wiki.syslinux.org/wiki/index.php?title=PXELINUX#config).

Another essential feature is multiboot, which is necessary for XCP-ng installations in UEFI environments. (Yes, UEFI is ok actually, just not work on my lab setup)

So, I'm faced with two challenges:

1. I need to determine the config paths (grub2.cfg).
2. I want to incorporate custom features into the grub2 image.

Regarding the first challenge, I've found a simple solution by examining the TFTP server's access log to identify the config path. This helps me locate the necessary config files for my setup.

For the second challenge, I've discovered that Debian's version of Grub can load configs based on MAC addresses, and I've learned that `grub2-mkimage` is the tool to create custom Grub images. 

However, it's important to note that we have to specify the required modules ourselves, and the module list can typically be obtained from the source code of the Grub package in the specific distribution.

You can check [this page](https://github.com/hugotkk/homelab-tftp/tree/main) to find out the commands to build the grub image.

## Lab Setup

The Ubuntu guide located at https://ubuntu.com/server/docs/install/netboot-amd64 has already covered 80% of the necessary steps for setting up various components, including PXE, the Apache server, and obtaining essential components such as the kernel, ramdisk, Grub2 EFI, and pxelinux.0.

Host System: Proxmox

OS to be netboot:

- Ubuntu 22.04
- Rocky 9.2
- XCP-ng 8.2
- Harvester v1.2.1

### Servers

- PXE Server: 203.0.113.182 (ubuntu22.04)
- Test VM: 203.0.113.100

PXE Server and Test VM are in the same subnet: 203.0.113.0/24

#### PXE Server

- Installed with dnsmasq for TFTP and DHCP services
- Apache2 server for hosting installation files

#### Test VM

- Attached with an empty disk
- Boot order set to disk first, then network
- Boot mode: UEFI

### Services

#### Dnsmasq

Dnsmasq is primarily used for DHCP - it assigns IP addresses to servers and maps these IP addresses to the MAC addresses of the VMs.

But it also serves as the tftp server

the conf of Dnsmasq be like:

```
interface=ens19,lo
bind-interfaces
enable-tftp
tftp-root=/srv/tftp

log-queries
log-dhcp
dhcp-leasefile=/var/dnsmasq/leasefile

dhcp-boot=pxelinux.0
dhcp-match=set:efi-x86_64,option:client-arch,7
dhcp-boot=tag:efi-x86_64,grubx64.efi
dhcp-range=ens19,203.0.113.2,203.0.113.254
dhcp-option=ens19,3,203.0.113.1
dhcp-host=BC:24:11:3A:C4:49,testvm,203.0.113.100
```

Breakdowns:

```
log-queries
log-dhcp 
```

I enabled logging to keep track of the dhcp / tftp requests for troubleshooting.

```
dhcp-boot=pxelinux.0
dhcp-match=set:efi-x86_64,option:client-arch,7
dhcp-boot=tag:efi-x86_64,grubx64.efi
```

This is a crucial setting. It directs the server to boot using `grubx64.efi` (GRUB2 EFI image) if it is in UEFI mode. In case of legacy mode, it uses `pxelinux.0`.

```
dhcp-option=ens19,3,203.0.113.1
```

This option is set to inform the VM that the gateway IP is 203.0.113.1.

```
dhcp-range=ens19,203.0.113.2,203.0.113.254
dhcp-host=BC:24:11:3A:C4:49,testvm,203.0.113.100
```

This specifies that any DHCP requests coming from `ens19` will have an IP lease range between `203.0.113.2` and `203.0.113.254`.

Furthermore, `dhcp-host=` is used to assign a specific IP (reverse 203.0.113.100) to a particular server, identified by its MAC address.

#### TFTP

TFTP server is used for serving the boot loader, its config (boot menu), kernel and ramdisk.

I opted for UEFI boot and chose GRUB2 as the boot loader.

The GRUB config file is located at `EFI/BOOT/grub.cfg`, and `grubx64.efi` is used as the GRUB EFI image.

For my XCP-ng installation, I had to switch to legacy boot due to some issues with UEFI.

In this scenario, `mboot.c32`, `menu.c32`, and `pxelinux.0` were utilized for the legacy boot process. The menu configuration for this is found in `pxelinux.cfg/default`.

Both the RAM disk and the kernel are extracted directly from the ISOs (`mount -o loop xxx.iso /mnt`) of the OS.

TFTP Server Folder Structure:
```
/srv

└── tftp
    ├── EFI
    │ └── BOOT
    │     └── grub.cfg
    ├── grubx64.efi
    ├── harvester-1.2.1
    │ ├── initrd
    │ └── vmlinuz
    ├── mboot.c32
    ├── menu.c32
    ├── pxelinux.0
    ├── pxelinux.cfg
    │ └── default
    ├── rocky-9.3
    │ ├── initrd
    │ └── vmlinuz
    ├── ubuntu-22.04
    │ ├── initrd
    │ └── vmlinuz
    └── xcp-ng-8.3.0
        ├── install.img
        ├── vmlinuz
        └── xen
```

grub.cfg
```
...

menuentry 'Ubuntu 22.04' {
    linuxefi /ubuntu-22.04/vmlinuz console=tty1 root=/dev/ram0 ramdisk_size=3000000 ip=dhcp url=http://192.168.0.182/iso/ubuntu-22.04.3-live-server-amd64.iso autoinstall ds=nocloud-net\;s=http://192.168.0.182/ubuntu/cloud-init/ cloud-config-url=/dev/null
    initrdefi /ubuntu-22.04/initrd
}

menuentry 'Rocky 9.3' {
    linuxefi /rocky-9.3/vmlinuz console=tty1 ip=dhcp inst.ks=http://192.168.0.182/rocky/ks.cfg inst.repo=http://192.168.0.182/rocky/repo
    initrdefi /rocky-9.3/initrd
}

menuentry 'xcp-ng 8.3.0' {
    multiboot2 /xcp-ng-8.3.0/xen dom0_mem=2048M,max:2048M watchdog dom0_max_vcpus=4 com1=115200,8n1 console=com1,vga
    module2 /xcp-ng-8.3.0/vmlinuz console=hvc0 console=ttyS0 answerfile=http://192.168.0.182/xcp-ng/answerfile.xml install
    module2 /xcp-ng-8.3.0/install.img
}

menuentry 'harvester 1.2.1 (Create)' {
    linuxefi /harvester-1.2.1/vmlinuz ip=dhcp net.ifnames=1 rd.cos.disable rd.noverifyssl console=tty1 root=live:http://192.168.0.182/harvester/rootfs.squashfs harvester.install.automatic=true harvester.install.config_url=http://192.168.0.182/harvester/config-create.yaml
    initrdefi /harvester-1.2.1/initrd
}

menuentry 'harvester 1.2.1 (Join)' {
    linuxefi /harvester-1.2.1/vmlinuz ip=dhcp net.ifnames=1 rd.cos.disable rd.noverifyssl console=tty1 root=live:http://192.168.0.182/harvester/rootfs.squashfs harvester.install.automatic=true harvester.install.config_url=http://192.168.0.182/harvester/config-join.yaml
    initrdefi /harvester-1.2.1/initrd
}

...
```

There are primarily two types of boot menu config to work with:

- Grub2 EFI: This config is intended for systems that use the UEFI standard for booting, representing a modern approach to the boot process (grub2.cfg).

- Legacy Boot: The legacy boot config is designed for older systems that lack UEFI support. It's crucial to have this config as a fallback to ensure compatibility with older hardware (pxelinux.cfg/default).

Both config serve the same purpose, which is to boot the kernel with a ramdisk, and they share similarities in their command structures.

For specific boot config and commands for each operating system, you can refer to their respective netboot documentation:

- [xcp-ng](https://docs.xcp-ng.org/installation/install-xcp-ng/#unattended-installation-iso-with-remote-config).
- [Ubuntu](https://ubuntu.com/server/docs/install/netboot-amd64).
- [Rocky](https://docs.centos.org/en-US/centos/install-guide/pxe-server/#sect-network-boot-setup-uefi), given its similarity to CentOS.
- [Harvester](https://docs.harvesterhci.io/v1.2/install/pxe-boot-install).

ChatGPT can be a valuable tool in converting these configurations from one format to another, such as from grub.cfg to pxelinux.cfg/default.

Furthermore, these commands can also be adapted for use with QEMU. This is especially useful for developing and testing auto-install config. 

For instance, you can simulate the installation process in a testing environment, as I did with Harvester, before deploying it in a live setting.

```
qemu-system-x86_64 \
-name harvester \
-enable-kvm \
-m 16384 \
-cpu host \
-smp 16,sockets=2 \
-hda /vms/harvester.img \
-nic user,hostfwd=tcp:127.0.0.1:1025-:22,dhcpstart=10.0.2.101 \
-bios /usr/share/ovmf/OVMF.fd \
-kernel harvester-v1.2.1-vmlinuz-amd64 \
-initrd harvester-v1.2.1-initrd-amd64 \
-append 'ip=dhcp net.ifnames=1 rd.cos.disable rd.noverifyssl console=ttyS0 root=live:http://192.168.0.182/harvester/harvester-v1.2.1-rootfs-amd64.squashfs harvester.install.automatic=true harvester.install.config_url=http://192.168.0.182/harvester/config-create.yaml' \
-no-reboot
```

#### Apache

Apache functions as an HTTP server responsible for delivering essential files required for OS installations.

When initiating an OS boot from the kernel, there is typically an option available to define auto-installation parameters, including:

- installation config (eg: keyboard, timezone, network, disk partition, post install)
- installation source (eg: cdrom or repo)

Config:

- Harvester uses [yaml](https://docs.harvesterhci.io/v1.1/install/pxe-boot-install/#preparing-ipxe-boot-scripts)
- Rocky, CentOS, and RedHat use (kickstart)[https://docs.centos.org/en-US/8-docs/advanced-install/assembly_kickstart-installation-basics/]
- Ubuntu uses [autoinstall](https://ubuntu.com/server/docs/install/autoinstall) config, which is a module of `cloud-init`
- XCP-ng uses [answerfile](https://docs.xcp-ng.org/appendix/answerfile/)

Source:

`Harvester` and `Ubuntu` are ISO-based, meaning the entire ISO is loaded into RAM. 

This requires the VM to have sufficient RAM to hold the entire content of the CD.

`XCP-ng` and `Rocky` installations use repository, which involves extracting contents from the ISO and copying them to Apache.

When copying files from the ISO to the Apache server, it's crucial not to overlook hidden files (dot files). These files can be essential for a valid repo.

Apache Server Folder Structure:
```
/var/www/html
.
├── harvester
│ ├── config-create.yaml
│ └── config-join.yaml
├── iso
│ ├── harvester-v1.2.1-amd64.iso
│ ├── openSUSE-Leap-15.0-NET-x86_64-Current.iso
│ ├── Rocky-9.3-x86_64-minimal.iso
│ ├── ubuntu-22.04.3-live-server-amd64.iso
│ ├── xcp-ng-8.3.0-beta1.iso
│ └── xcp-ng-8.3.0-beta1-netinstall.iso
├── README.md
├── rocky
│ ├── ks.cfg
│ └── repo
│     ├── BaseOS
│     ├── EFI
│     ├── images
│     ├── isolinux
│     ├── LICENSE
│     ├── media.repo
│     ├── minimal
│     └── rocky.img
├── ubuntu
│ └── cloud-init
│     ├── meta-data
│     └── user-data
└── xcp-ng
    ├── answerfile.xml
    ├── noresync.sh
    ├── post-install-script
    └── repo
        ├── boot
        ├── EFI
        ├── EULA
        ├── install.img
        ├── LICENSES
        ├── Packages
        ├── repodata
        └── RPM-GPG-KEY-xcpng
```

