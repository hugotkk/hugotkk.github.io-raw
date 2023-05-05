---
title: "Ubuntu 22.04 autoinstall"
date: 2023-05-04
tags:
- ubuntu
---

Working with Ubuntu 22.04 autoinstall can be challenging due to incomplete documentation and time-consuming debugging. This post shares my experience and offers tips to help others facing similar issues.

Here is the documentation of [autoinstall](https://ubuntu.com/server/docs/install/autoinstall-quickstart) and [curtain](https://curtin.readthedocs.io/en/latest/index.html).

And my work: https://github.com/hugotkk/labs/blob/main/ubuntu-quickstart/user-data

## Remember the #cloud-config Header

The `#cloud-config` header is crucial for a successful autoinstall. Without it, the process won't work.

## Customizing Storage: Fixing the 'fstype' Error

While customizing storage, I encountered an error: "LVM_LogicalVolume object has no attribute 'fstype'." To resolve this, modify the 'mount' reference to 'format' instead of LVM, as shown below:

```
      - id: lv-root
        type: lvm_partition
        name: root
        volgroup: vg
        size: 10G
      - id: root-fs
        type: format
        fstype: ext4
        volume: lv-root
      - id: mount-root
        type: mount
        path: /
        device: lv-root # should reference to root-fs not lv-root
```

## Installing Software in late-commands

When trying to install software in the `late-commands` section, keep in mind that during autoinstall, Ubuntu is mounted on `/target`, not `/`. To address this, use curtin:

```
late-commands:
- curtin in-target --target=/target -- <INSTALL_CMD_LIKE_APT_INSTALL>
```

## Two Ways to Use autoinstall

There are two methods for using autoinstall: over HTTP and with an additional volume. I found the latter more practical in my case as I'm using Virtualbox.

