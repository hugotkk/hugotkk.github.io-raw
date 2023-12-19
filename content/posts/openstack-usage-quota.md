---
title: "Openstack - Usage & Quota"
date: 2023-12-19
tags:
- openstack
- quota
---

## List Compute

```bash
openstack host list
openstack hypervisor list
openstack compute server list
```

## List Services

```bash
openstack server list
openstack endpoint list
```

## Compute usage

eg: RAM, CPU, DISK

```bash
openstack usage list
openstack hypervisor stats show

```
## Quota usage

```bash
mysql
```

```sql
use nova;
select * from quotas_usage;
```

```bash
mysql
```

```sql
use neutron;
select * from quotas_usage;
```

## Set Quota

Per Project
```bash
openstack quota show <project>
openstack quota set <project> --<quota-key> <quota-value>
```

Global default
```bash
openstack quota set default --class
```

