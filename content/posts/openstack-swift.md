---
title: "Openstack - Swift"
date: 2023-12-19
tags:
- openstack
- swift
---

## Create swift ring

Swift is built on three primary components:

1.  Object

2.  Account

3.  Container

For each of these components, ring files are required:

-   <component>.builder

-   <component>.ring.gz

Ring files store information about the disks in the Swift storage system. When a new disk is added or an existing disk undergoes changes, it's essential to regenerate and rebalance the relevant ring file.

For instance, when creating an object ring:

```bash
swift-ring.builder object-1.builder create 10 3 1
```

10: This represents the partition power, implying that the ring will be divided into 2^10 partitions.

3: Indicates the replica count. With a value of 3, the data will be replicated into three copies.

1: Refers to min_part_hours. It ensures that when a disk goes out of service or a new disk is added, only one data piece will move to a new partition in an hour. This mechanism ensures a minimum of two copies are always available during any operation.

The matrix formed from the partition and replica information indicates where the second and third copies of each data piece will be stored.

The system detects the failure and identifies the data that was on the failed disk.

Temporary partitions are created on other disks or nodes to take over the role of the failed disk.

The data that was on the failed disk is replicated to these temporary partitions to restore the desired level of redundancy.

Adding disks to the Ring:

```bash
swift-ring-builder object-1.builder add <disk> <weight>
```

```bash
swift-ring-builder object-1.builder add r1z1-127.0.0.1:6001/d1 1
```

Here, parameters like z for zone, r for region, and other parameters related to host and disk provide constraints on where copies of the data are stored.

d1 is the disk label, 1 is the weight

Swift intelligently places replicas as far apart as possible to ensure data durability. For instance:

If there are multiple zones, regions, or hosts, Swift tries to keep replicas in different zones, regions, or hosts.

If there's only one zone, region, and host, Swift will ensure that replicas are on different disks within that host.

<https://docs.openstack.org/kolla-ansible/latest/reference/storage/swift-guide.html>

## Create storage policy

Storage policies start from an index of 0. The first ring corresponds to [storage-policy-0], the second ring to [storage-policy-1], and so on.

Each policy can be used to specify a different ring setting, which in turn can be used during container or object creation.

Example:

```
[storage-policy:0]
name = gold
default = yes

[storage-policy:1]
name = silver
```

Add the above policy definitions to the /etc/kolla/config/swift.conf file.

Redeploy the openstack

```
kolla-ansible -i ./all-in-one deploy
```

To create container with storage policy:

```bash
openstack container create <container> --storage-policy <storage-policy>
```

## Set Object Expiration Date

```bash
swift post <container> <object> -H "X-Delete-After:<seconds>"
swift post <container> <object> -H "X-Delete-At:<timestamp>"
```

Timestamp can be obtained from

```bash
date +%s -d "tomorrow 00:00"
```

## Remove Object Expiration Date

```bash
swift post <container> <object> -H "X-Remove-Delete-At:"
```

## ACL

```bash
Swift post -r "<read acl>" -w "<write acl>
```

| Permission       | Pattern        |
|------------------|----------------|
| read only bucket | `r.*`, `.rlistings` |
| project          | `<project>:*`  |
| user             | `*:<user>`     |
| role             | `<role>`       |


## Download Object

```
openstack object save <container> <object> --filename <file>
```

