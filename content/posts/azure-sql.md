---
title: Azure SQL Server
date: 2024-07-16
tags:
  - azure
  - sql
---

Data Encryption (PII) = Always Encrypted

## Key Considerations

- For HA: Business Critical or Hyperscale
- For cost-effective HA solution: General Purpose
- For multiple databases with varying workloads: Elastic Pool
- For SQL Server features not available in Azure SQL Database: Managed Instance
- Azure Synapse Analytics is optimized for data warehousing and analytics workloads, not for OLTP scenarios
- Azure Hybrid Benefit for SQL Server can only be applied to the vCore-based purchasing model

### Zone Redundancy

Supported
- General Purpose (SQL Database and Managed Instance)
- Business Critical
- Hyperscale
- Premium
Not Supported
- Basic
- Standard

### Cost

(from lower to higher)

- Serverless
- Premium
- Business Critical
- Hyperscale

### Azure SQL Database

#### Hyperscale

- Supports up to 100TB storage
- Most expensive and powerful option
- Only tier that can scale up and down seamlessly
- 0 - 4 readable replicas

#### Business Critical

- Offers auto-failover, low latency, high availability
- Remains available even if two availability zones fail within a region
- 3 replicas, 1 readable

#### Basic / Standard

- no Zone redundancy support

#### Elastic Pool

- Suitable for multiple databases with varying usage patterns
- Allows sharing of resources (CPU, storage)
- Can scale dynamically

#### Serverless

- Only available in vCore-based purchasing models
- Supports General Purpose and Hyperscale (not Business Critical)

### Azure SQL Managed Instance

- Supports General Purpose and Business Critical tiers
- Provide zone redundancy support
- Supports up to 16TB storage
- Support auto failover group (server level)
- Good for specific features like cross-database transactions, TSQL

### SQL Server on Azure VM
- Provides full control over SQL Server engine
- Not recommended if SLA is a requirement

## HA and DR Options

### Azure SQL Database

**Auto-Failover Group (Server-Level)**

- Provides automatic failover at the server level
- Can be configured within the same region or across different regions
- Supports read-write and read-only endpoints

**Active Geo-Replication (Database-Level)**

- Provides database-level replication to a different region
- Must be configured across regions
- Allows up to four replications
- Suitable for DR and read scalability

SQL Managed Instance supports only auto-failover groups for HA.

### SQL Server

**Failover Cluster Instances (Server-Level)**

- Requires shared storage (e.g., SAN).

**Availability Groups (Database-Level)**

- Supports multiple databases within a group.
- Can be configured with Distributed Network Name (DNN) for multiple subnets or Virtual Network Name (VNN) for a single subnet.
- DNN minimizes failover time and costs in multi-subnet configurations.

## Audit Log 

### Retention

- The maximum retention period is 2 years
- The minimum retention period is 90 days

### Storage

- Block Blob: 
  - Standard storage account v2 (block blobs)
  - Premium block blob storage
- The storage account should be in the same region as the SQL server
- For servers behind a VNet or firewall, standard storage account v2 is required

## Backups

### Point-in-Time Backups

- Retention Period is 7-35 days

### Long Term Backups (LTB)

- LTB stored in Azure Blob storage.
- The maximum retention period is 10 years