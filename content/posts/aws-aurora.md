---
title: "Aurora"
date: 2022-02-21
tags:
- cloud
- aws
- devops
- aurora
- database
---

## RDS Read Replica

* cross AZ: OK
* cross region: OK
* **promote / point-in-time recovery / restore from snapshot** will create a new db instance which is not in my expectation. When DR, user need to update dns / db endpoint in application

### Aurora

#### Pros over non-aurora rds

* read replica auto scaling
* failover (failover master to read replica in same region)
* multi-az by default
* endpoint for writer and reader instance (also custom endpoint)

#### Cons in aurora

##### Bad integration in cross-region read replica

For example:

* Region1: master, read-replica
* Region2: master, read-replica

The replication between the multi region db clusters is:
* master-slave replication between region1 master and read-replica
* master-slave replication between region2 master and read-replica
* master-slave replication between region1 master and region2 master

When write record to region1 master, the record will be replicated to 
* region1 read-replica (by region1 master)
* region2 master (by region1 master)
* region2 read-replica (by region2 master)

When writing a record to region2 master, the record will only be synchronised to region2 read-replica. 

And both region1 and region2 masters are writables!!!

AWS "assume" people will use the region2 cluster as a readonly database. 

This is not mentioned in aws doc. I found the [answer](https://repost.aws/questions/QUrCbnj0u4TWaz-A1uR-QDPQ/aurora-create-cross-region-read-replica-vs-add-region) from aws:repost instead.

* when create a cross-region read-replica, it will create another db cluster in the new region
* you cannot see the cross-region db in the same aws console
* endpoint cannot cross region, only within the region
* need to manually set binlog_format in the parameter group to before creating the read replica
* need to manually set read_only true in the parameter group at second region db cluster to prevent its writer instance from writable by other
* endpoint writer / reader instance depends on role so the writer endpoint in second region is incorrect and should be not used
* route53 + latency based routing policy will need to be set up to route both reader endpoints to utilise those read-replicas. but the reader endpoint does not include the master. if you want to utilise the second region master, You need to create a custom endpoint for this

So AWS created [Aurora Global Database](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database-connecting.html) to solve these drawbacks. But it does not support on small db instance type!

### aurora serverless

* [features](https://aws.amazon.com/rds/aurora/pricing/)

#### Pros

* scale up quickly (v1: 1min, v2: immediately)
* scale down quickly (v1: 15min, v2: 1min)
* save cost - can scale down to 0 when no active traffic 
* MySQL & PostgreSQL compatible
* no need to manage the cluster

#### Cons

* price 30% higher than same power's ec2 instance
* cannot cross-region
* no public access
