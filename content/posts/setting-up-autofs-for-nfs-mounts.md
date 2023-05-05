---
title: "Setting Up Autofs for NFS Mounts"
date: 2023-05-05
tags:
- nfs
- autofs
---

Set up NFS:
- Follow this guide (for ubuntu): https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-20-04

Autofs is a file system management tool that offers several key features:

- Dynamic Mounting: File systems are mounted on-demand when they are accessed, reducing the number of mounts that need to be managed manually.
- Improved Performance: By reducing the number of mounted file systems, Autofs can improve system performance by reducing the amount of system resources required to manage mounts.
- Increased Security: Autofs can increase system security by reducing the number of mounted file systems available at any given time. This reduces the attack surface of the system.

Autofs mount methods:
- Direct
- Indirect

Let's say nfs server is 192.168.0.101.

Direct mapping example:
- Map 192.168.0.101:/data to /data

```
vi /etc/master.auto
```

```
/- /etc/direct.auto
/home /etc/home.auto
```

```
vi /etc/direct.auto
```

```
/data 192.168.0.101:/data
```

Indirect mapping example:
- Map 192.168.0.101:/data/hugotse to /home/hugotse
- Map 192.168.0.101:/data/<any> to /home/<any>

```
vi /etc/home.auto
```

```
hugotse 192.168.0.101:/data/hugotse
* 192.168.0.101:/data/&
```

Autofs man page:
- https://manpages.ubuntu.com/manpages/focal/en/man5/autofs.5.html
