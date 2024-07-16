---
title: Azure Migration
date: 2024-07-16
tags:
  - azure
  - azure migrate
  - migration
  - vmware
  - sql
---

## Azure Migrate

Azure Migrate can be using for planning and executing VMware migrations to Azure

- Can give migration plan for VMWare Migration
- Discover and assess on-premises VMware VMs
- Provide sizing and cost estimation for Azure VMs
- Using Azure Site Recovery under the hood for the actual replication and migration process

## SQL Server Migration

- **Azure Database Migration Service** (via Azure Data Studio) is best for large-scale migrations with minimal administrative effort
- **Data Migration Assistant** is a good second choice for more controlled, smaller migrations
- **SQL Server Migration Assistant** is specifically for migrating from non-SQL Server databases

### Azure Database Migration Service (DMS)

- Integrated into Azure Data Studio via the Azure SQL Migration extension
- Supports migrating SQL Server databases to:
  - Azure SQL Managed Instance
  - SQL Server on Azure VM
  - Azure SQL Database (offline mode only)
- Offers online (minimal downtime) and offline migration modes
- Provides assessment, SKU recommendations, and orchestrates the migration process

### Data Migration Assistant (DMA)

- Standalone tool for assessing and migrating SQL Server databases
- Good for smaller migrations
- Can migrate to Azure SQL Database, Azure SQL Managed Instance, and SQL Server on Azure VM
- Provides compatibility assessment before migration

### SQL Server Migration Assistant (SSMA)

- Used for migrating non-SQL Server databases (e.g. Oracle, DB2, MySQL) to Azure SQL
- Converts schema and migrates data from other database platforms

### Using Always On & Distributed Availability Groups

Can be used for cross platform migration (from Windows to Linux SQL Server)

- Set up an Always On Availability Group on Windows SQL Server
- Add the databases to the availability group
- Create a Distributed Availability Group that spans both the Windows and Linux SQL Servers

- The Distributed Availability Group will synchronize data between the Windows and Linux SQL Servers
- Once synchronization is complete, we can failover to the Linux SQL Server



