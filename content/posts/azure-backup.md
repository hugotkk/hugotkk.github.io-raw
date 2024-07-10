---
title: Azure Backup
date: 2024-07-06
tags:
  - azure
  - recovery service
  - backup service
---

## Recovery Service Vault

The Vault needs to be in the same region as the VM.

The vault includes two primary services:
1. Backup Service
2. Recovery Service (for Disaster Recovery)

Backup:
- Typically has longer Recovery Point Objective (RPO) and Recovery Time Objective (RTO)
- Backup frequency options:
  - Hourly (every 4 hours)
  - Daily
  - Weekly
- Longer RPO generally means more data to back up, which can lead to a longer RTO
- Supports long retention periods, up to 10 years

Recovery:
- Primarily used for Failover and Disaster Recovery (DR) scenarios
- Usually has shorter RPO and RTO (0-12 snapshots in hours)
- Supports recovery between:
  - Azure to Azure (different regions)
  - On-premises to Azure
  - Azure to on-premises

### Backup

There are three items that can be backed up:

- VM
- SQL on VM
- File Shares

### Steps to Create a Backup on VM:

1. Create a Recovery Services Vault
2. Configure Storage Replication Type to ZRS
3. Create a Backup Policy
   - Define the schedule
   - Set retention rules
4. Configure Backup for VM

### Backup Report

We can configure backup report at Diagnostic setting.

There are two main options to store the data:
- Storage account 
  - Must be in the same region as Vault
  - Helps minimize latency
- Log analytics 
  - Can be in a different region from Vault
  - allows for centralized logging across multiple regions

### Backup Agent 

When configuring the backup on a VM, a backup agent will be deployed to the VM.

For Windows VM,
- The Recovery Services Agent (MARSA) is used.

For Linux VM,
- The Azure VM Backup extension is used.

### Cross Region Restore (CRR)

CRR is an opt-in feature that allows data recovery in a secondary region if the primary region is unavailable.

It needs to be enabled at the vault level.

Once enabled, all subsequent backups are automatically replicated to the secondary region.

Supported Backup Types for CRR:
- MARS agent backups
- Azure VM
- Azure Files share

### Backup Policy

- Backup policies cannot be shared between different Recovery Services Vaults.
- Each vault have its own backup policy. (can't share between vault)

**Enhanced Policy Features:**
- Supports multiple backups per day
- 30-day tier retention
- Supports trusted launch VMs
- Supports SSH v2 and Ultra Disk

**Standard Policy Features:**
- Supports one backup per day
- 5-day tier retention

### Restore backup

Restore backup to existing vm, it will NOT overwrite the
- instance size change
- admin password change

And
- data disk will become unattached
- data on tmp disk (D:\ by default) will be lost
- data on OS disk will be restored

To restore a file
- Click File Recovery (vs Restore VM)
- Select a restore point
- Download and run the script that mounts the disk from the selected recovery point as local drive
- Copy the desired file using file explorer

### Site Replication (DR)

To Replicate On-Premise Servers to Azure:

Physical Host:
1. **Create Azure Recovery Services Vault**
2. **Install Appliance and Register the Server**
3. **Create Storage Account**
4. **Configure Replication Policy**
5. **Enable Replication**

Hyper-V Server:
1. **Create Azure Recovery Services Vault**
2. **Create Hyper-V Site**
3. **Install "Site Recovery Provider" and Register the Server into the Hyper-V Site**
4. **Configure Replication Policy (e.g., interval)**
5. **Enable Replication**

When replicating a VM using Site Recovery:

- The VM will replicate to the subnet with the same name as the source subnet.
- If no matching subnet name is found, it will replicate to the first subnet in the target network, sorted in alphabetical order.

### Delete Vault

1. **Stop Backup**
2. **Disable Soft Delete and Immutable Vault**
3. **Delete Backup Data**
   - SQL
   - VM
   - File Share
4. **Remove Linkage of Resources and Storage Account**
5. **Stop Replication**

## Backup Vault

Recovery Service Vault supports VM, SQL on VM and File Share.

While Backup Vault supports:
  - Disk
  - Blob
  - PostgreSQL / MySQL
  - AKS

### Steps to Create a Backup on Disk:

1. **Create a Backup Vault**
2. **Create a Backup Policy**
   - Define Schedule
   - Set Retention
3. **Choose Policy**
4. **Add Data Sources (Disk)**
   - At this stage, we can click "assign missing roles" to add necessary roles to the disk

### Required Roles
- **Disk Reader**
- **Snapshot Contributor**

## Backup at App Service

Setting retention to 0 days means the backup is kept indefinitely.

The backup of a web app includes only the production slot. We cannot restore a test slot from backup.

The storage account used for backing up in App Service must be a Blob Storage.