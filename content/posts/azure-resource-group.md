---
title: Azure Resource Group
date: 2024-07-06
tags:
  - azure
  - resource group
---

## Resource Mover

We can move a resource between resource groups within the same subscription or to a different subscription. 

The resource's location will stay the same during the move. 

However, Azure policies might change since they depend on the subscription, resource group, or management group the resource is part of.

Supported Resources:
- VM
- VNET, NSG, IP, NICS
- Managed Disks
- Availability Sets
- App Service Plan and App
- Storage Accounts
- Recovery Services Vaults
- SQL
- Cosmos DB

Unsupported Resources:
- Load Balancer
- AKS
- Domain Services
- Batch accounts
- Redis
- Data Factory
- ExpressRoute circuits

## Deletion

There are two common factors that prevent the deletion of a resource group:
1. Resource Lock
2. Recovery Services Vault

For Resource Lock,
- Check for locks at both the Resource Group and Recovery Service Vault
- Remove any existing locks

For Recovery Service Vault:
- **Backup Items (including soft-deleted items)**: Stop the Backup at Backup Item. Diable soft delete. Delete backup data.
- **Protected Resources**: Stop protection on VMs
- **Linked Storage Accounts**: Unregister any linked storage accounts under Backup Infrastructure > Storage Account
- **Private Endpoints**: Remove any associated private endpoints.
- **Replication Data**: Stop replication and remove any replicated items.
