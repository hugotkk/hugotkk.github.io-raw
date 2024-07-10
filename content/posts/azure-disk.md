---
title: Azure Disk
date: 2024-06-26
tags:
  - azure
  - disk
---

## Disk Type

### SSD
- Standard SSD: General Purpose
- Premium SSD: Support high IOPS
- Ultra Disk: High-performance option (Overskill for most scenarios)

### HDD
- Standard HDD

## Disk Caching

Azure provides three caching options for disks:

- **Write-only**: Caches data only for write operations.
- **Read-only**: Caches data only for read operations.
- **Read and write**: Caches data for both read and write operations.

For SQL Server workloads:
- **Log files**: No caching is recommended.
- **Data files**: It is recommended to use read-only caching.

For scenarios where preventing data loss is critical, read-only caching is the safest option.

## Disk Encryption

Azure offers three primary methods for disk encryption:

1. **Server-Side Encryption (SSE)**
   - Default encryption method managed by Azure Storage.
   - Disk is decrypted when exported to VHD.
   - More advanced configurations can be applied via Disk Encryption Set.

2. **Azure Disk Encryption (ADE)**
   - Uses DM-Crypt for Linux or BitLocker for Windows.
   - Can be enabled via VM extension or in "Additional Settings" under "Disks".
   - Disk remains encrypted when exported to VHD.
   - Decryption requires the Key Vault key.

3. **Encryption at Host**
   - Requires support in the subscription plan.
   - Encrypts data from the VM to Azure Storage.

## Disk Encryption Set

Disk Encryption Set (DES) offers three types of encryption:

1. Encryption at rest with Customer-Managed Key (CMK)
2. Confidential Encryption with CMK
3. Double Encryption with CMK + Platform-Managed Key (PMK)