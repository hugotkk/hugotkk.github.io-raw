---
title: Azure Storage & Key Vault
date: 2024-06-26
tags:
  - azure
  - storage account
  - key vault
---

## Quick Summary

| Service        | File Storage | Blob Storage |
|----------------|--------------|--------------|
| AzCopy         | Yes          | Yes          |
| Import/Export  | Yes          | Yes          |
| File Sync      | Yes          | No           |

| Auth\Tool    | AD   | Account Key  | SAS  |
|--------------|------|------ |------|
| SMB          | Yes  | Yes   | No   |
| AzCopy       | Yes  | Yes   | Yes  |
| CLI          | Yes  | Yes   | Yes  |

## Storage account 

Account types sorted by timeline:

1. **StorageV1:** Does not support Blob features, leading to the introduction of BlobStorage.   
2. **BlobStorage:** Introduced to provide Blob-specific capabilities.
3. **StorageV2:** Includes support for ZRS.
4. **Data Lake Gen2:** Supports Hierarchical Namespace, which provides a real directory structure and POSIX ACL capabilities.

**Key Features and Limitations:**

- **StorageV1:** Does not support Blob features.
- **ZRS:** Not supported in StorageV1.
- **GRS:** Not supported in Premium storage due to its focus on low latency.
- **Premium:**: Does not support Tier Access. Premium aimed at low-latency, high-performance needs rather than long-term storage.

**Tier Access:**
- **Hot / Cool / Cold / Archive:** Provides varying levels of access and cost-efficiency. 
- **Archive tier**:
  - requires up to 15 hours to transition from offline to online tiers (hot / cool).
  - Does not support in any Zone related redudancy, as it is designed for archival purposes, not HA.

**Blob Features** (supported in all storage accounts except V1):
1. **Tier Access:** Enables cost savings by managing data storage in different tiers (hot, cool, cold, archive).
2. **Lifecycle Management:** Facilitates moving objects between tiers and setting retention periods for data.
3. **Access Policies:** Allows setting up to 5 access policies to limit SAS.
4. **Immutable Policies:** Supports configuring up to 2 immutable policies to prevent data modification or deletion.
5. **Object Replication:** Provides options to replicate objects to specific region. GRS does not allow choosing the replication region.
6. **Scoped Encryption:** Allows using different encryption keys for specific containers.

- **IAM Role Support Conditions:**
  - Containers
  - Queues

- When using Azure Backup for VMs or Azure Site Recovery with a storage account that has restricted public access, enabling "Allow trusted Microsoft services to access this storage account" is necessary.

- Live migration of storage account redundancy facilitates the transition from LRS to ZRS.

- ZRS ensures data resilience in the event of a regional data center failure.

### SMB

When authenticating with Active Directory (AD) while using SMB in File Share, Azure Domain Services can be utilized.

### SAS (Shared Access Signature)

A SAS is an access token that provides temporary permissions to the storage account.

It allows fine-granted control over which resources can be accessed and what actions can be performed on them.

Resources are categorized into:
- Service
- Container
- Object

Permissions that can be granted include:
- READ
- WRITE
- DELETE
- LIST
- ADD
- CREATE

| Operation                  | Required SAS Permissions               | Command                                                                                               |
|----------------------------|----------------------------------------|-------------------------------------------------------------------------------------------------------|
| **Download Blob**          | Resource Type: Object <br> Permissions: Read | ```az storage blob download --account-name "$account" --container-name "$container" --sas-token "$token" --name "$file"``` |
| **List Blobs in Container**| Resource Type: Containe <br> Permissions: List | ```az storage blob list --account-name "$account" --container-name "$container" --sas-token "$token"``` |
| **List Containers**        | Resource Type: Servic <br> Permissions: List | ```az storage container list --account-name "$account" --sas-token "$token"``` |

### Access Policy

A SAS is signed using the account key. 

If a SAS is compromised, it cannot be revoked directly. 

One of the solution is to rotate the account key, but this affects all SAS tokens signed with it. 

Alternatively, we can use an access policy to further restrict the SAS.

Each container can have up to `5` access policies defined.

### Immutable Blob Storage Policy

An immutable blob storage policy can define up to 2 policies:

1. Legal Hold: This policy allows creating and reading blobs but prevents them from being modified or deleted.
2. Time-Based Retention: This policy allows WORM (writing once and reading many) within a specified period. After this period, the object can be deleted but not modifiedn.

### Lifecycle Management

Lifecycle Management applies rules specifying blob types
- BlockBlob
- AppendBlob

and their subtypes
- base
- snapshot
- version

```
{
  "rules": [
    {
      "name": "exampleRule",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "version": { # this rule apply to version only
            "delete": {
              "daysAfterCreationGreaterThan": 30
            }
          }
        },
        "filters": {
          "blobTypes": [
            "blockBlob" # rule apply to BlockBlob
          ]
        }
      }
    }
  ]
}

```

If multiple rules apply, the less expensive action is chosen:
- `delete` > `move to archive` > `move to cool`.
- Objects moved to archive are not moved back to cool.

**Filters:**
- `prefixMatch`: Matches file paths.
- `blobIndexMatch`: Matches index tags (a property of objects).
- `blobTypes`: Specifies block or append blob.

**Actions:**
- For `baseBlob`:
  - `enableAutoTierToHotFromCool`: Enables switching files back to hot if accessed.
- `tierToArchive`: Moves files to archive.
- `tierToCool`: Moves files to cool.
- `tierToCold`: Moves files to cold.
- `delete`: Delete file.

### IAM with Conditions

To restrict view permissions to specific blobs, IAM can be configured with conditions based on blob index tags.

**Example:**

- Role: Storage Blob Data Reader
- Action: Read (to view blobs)
- Condition Expression:
  - Attribute source: Resource
  - Attribute: Blob index tags `Values in key`
  - Key: `<Key>`
  - Operator: StringEquals
  - Value: `<Value>`

### Encryption

Customer-managed keys:
- Use RSA
- Support key sizes of 2048, 3072, and 4096 bits
- Stored in Azure Key Vault or Azure Managed HSM
- Allow for key rotation and revocation managed by the customer

Azure-managed keys:
- Use symmetric key encryption (AES-256)
- Managed entirely by Microsoft
- Automatically rotated by Microsoft
- Require no setup or management by the customer

### Network ACL

**Example Configuration:**

- **Bypass:** AzureServices
- **Virtual Network Rules:** None configured
- **IP Rules:** None configured
- **Default Action:** Allow

This configuration indeed allows all access by default. The "AzureServices" bypass allows trusted Microsoft services to access the storage account even if network rules are in place.

When no virtual network or IP rules are specified, and the default action is set to "Allow", public IP access is permitted from any network.

**Network Modes:**

1. **Enable Public Access:** 
   - **Default Action:** Allow
   
2. **Enable Access from Selected Virtual Networks and IP Addresses:**
   - **Firewall (IP Rules)**: Allows us to specify individual IP addresses or CIDR ranges that can access the storage account.
   - **Virtual Network (Virtual Network Rules)**: Allows us to specify Azure Virtual Networks that can access the storage account.
   - **Exceptions (Bypass)**: Allows us to bypass the firewall for certain Azure services.

3. **Disable Public Access:**
   - **Default Action:** Disabled
   - This is the most restrictive setting, blocking all public access. Access is only possible through specified virtual networks or IP addresses.

### Azure File Explorer

Capability:
- Cannot create storage accounts.
- Can change blob types during file uploads, choosing from append, block, or page.
- Can create file shares, containers, queues, and tables.
- Can also add data to tables and queues.

## File Sync

Similar to Google Drive, File Sync allows us to synchronize folders on Windows with a file share.

Steps:
1. Create a storage sync service.
2. Install the File Sync agent on Windows.
3. Register the server.
4. Create a sync group and cloud endpoint.
5. Add server endpoints.

- Each sync group can:
  - Have one cloud endpoint (file share).
  - Be registered with multiple servers.
  - Have one server endpoint per server. For example, a server endpoint is a local file path like `D:\data`. You cannot add both `\\Win0001:\C$\data1` and `\\Win0001:\C$\data2` to the same sync group, but different servers are allowed.
- Each sync group can only have one cloud endpoint.
- Only servers that are registered can join the sync group.
- Uploading files to the file share can cause conflicts: newer files replace older ones, which will be renamed.
- Changes made to files on the file share are detected by Azure File Sync once every 24 hours by default. We can manually trigger a sync to update changes sooner.
- Changes made locally are updated in real-time.

## Import/export

Azure Import/Export is akin to AWS's Snow Family, facilitating the transfer of vast data from on-premises to the cloud by shipping data on disks to Azure.

**Setup Steps:**

1. **Prepare Data and Drives:**
   Before copying data to disks, prepare:
   - **Dataset CSV:** Lists files and their corresponding storage account mappings.
   - **Driveset CSV:** Details about the disks.

2. **Copy Data to Disk and Create Journal File:**
   Execute the following command to initiate data copying and create a journal file:
   ```
   .\WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#1 /InitialDriveSet:driveset.csv /DataSet:dataset.csv /logdir:C:\logs
   ```
   This command prepares for the import job and logs the process.

3. **Upload Journal File and Create Import Job:**
   After data copying completes, a journal file is generated. Create import job with necessary details. Upload this file to it. 

4. **Ship Disks to Azure:**
   Physically transport the disks containing the data to Azure’s designated facility.

5. **Update Import/Export Information:**
   Inform Azure that the disks have been shipped. This step finalizes the data transfer process.

**Other Data Migration Options:**

- **Import/Export:** Suitable for up to 1TB of data.
- **Data Box Disk:** Supports up to 35TB of data.
- **Data Box:** Handles up to 80TB of data.
- **Data Box Heavy:** Capable of managing up to 800TB of data.

These options provide flexibility depending on the data volume and transfer requirements.

## azcopy

tools to copy data to storage account.
support file / blob
can auth with AD / SAS
in the past, file share only support auth with SAS

azopy support on 
Windows
Linux
Macos

## Disk

### Disk Caching

Azure provides three caching options for disks:

- **Write-only**: Caches data only for write operations.
- **Read-only**: Caches data only for read operations.
- **Read and write**: Caches data for both read and write operations.

For SQL Server workloads:
- **Log files**: No caching is recommended.
- **Data files**: It is recommended to use read-only caching.

For scenarios where preventing data loss is critical, read-only caching is the safest option.

### Disk Encryption

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

### Disk Encryption Set

Disk Encryption Set (DES) offers three types of encryption:

1. Encryption at rest with Customer-Managed Key (CMK)
2. Confidential Encryption with CMK
3. Double Encryption with CMK + Platform-Managed Key (PMK)

## Key Vault

To enable volume encryption using the Key Vault, navigate to Resource Access and enable "Azure Disk Encryption for volume encryption".

To reference secrets stored in Key Vault within an ARM Template, it is essential to enable "Access Azure Resource Manager for template deployment".

To integrate Key Vault with an App Service:
- Create a Managed Identity for App.
- Configure IAM settings in Key Vault.
