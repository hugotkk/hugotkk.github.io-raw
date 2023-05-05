---
title: "Managing Disk Space with LVM: A Comprehensive Guide"
date: 2023-05-05
tags:
- linux
- lvm
- disk
---

Working with LVM (Logical Volume Management) can simplify the process of managing disk space. This post will guide you through enlarging a disk in LVM, migrating a non-LVM file system to LVM, and migrating the root file system to LVM.

## Enlarging a Disk in LVM

Refer to this article for more information: [Manage and Create LVM Partition Using vgcreate, lvcreate, and lvextend](https://www.tecmint.com/manage-and-create-lvm-parition-using-vgcreate-lvcreate-and-lvextend/).

To enlarge an existing LVM:

1. Check if the new disk is detected using `lsblk`. If not, scan it with:

```
echo "- - -" > /sys/class/scsi_host/host<N=1,2,3,4...>/scan
```

2. Create a physical volume, extend the volume group, and extend the logical volume:

```
pvcreate <new_disk>
vgextend <vg> <new_disk>
lvextend -l +100%FREE -r <lvm>
```

3. Resize the file system:
   - For ext4, use [resize2fs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/recognize-expanded-volume-linux.html#extend-file-system).
   - For xfs, use [xfsgrow](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/storage_administration_guide/xfsgrow).

4. You should able to see the resize disk on

```
df -h
```

## Migrating Non-LVM File System to LVM

For example, if you want to move `/var` to LVM:

1. Create the LVM.
2. Mount it to a temporary location (e.g., `/tmp/lvm`).
3. Copy data from `/var` to the LVM:

```
rsync -avz /var/ /tmp/lvm/
```

4. Unmount it.
5. Update `/etc/fstab` with the new LVM.
   - Use UUID to reference the file system.
   - Find the UUID with [blkid](https://manpages.ubuntu.com/manpages/focal/man8/blkid.8.html).

Refer to [fstab documentation](https://manpages.ubuntu.com/manpages/focal/en/man5/fstab.5.html) for more details.

## Migrating Root File System to LVM

Refer to [Converting an Existing Root Filesystem to LVM Partition](https://www.thegeekdiary.com/centos-rhel-converting-an-existing-root-filesystem-to-lvm-partition/).

The steps are similar to migrating a non-LVM file system:

1. Repeat the steps of migrating Non-LVM File System
5. Perform additional steps:
   - `chroot` to update `initrd`.
   - Edit `grub.conf` with the new root.
6. Restart the system.


Following these steps will help you manage disk space using LVM, whether you need to enlarge a disk, migrate a non-LVM file system to LVM, or migrate the root file system to LVM.