---
title: Azure Storage Account
date: 2024-07-10
tags:
  - azure
  - storage account
---

## Evolution

- **StorageV1:** Original
- **BlobStorage:** Introduced for Blob-specific capabilities
- **StorageV2:** Added ZRS support
- **Data Lake Gen2:** Supports Hierarchical Namespace, POSIX ACL, HDFS protocol

## Limitations

- **StorageV1:** No Blob features, no ZRS support
- **ZRS:** Not in StorageV1.
- **GRS:** Not in Premium storage, focused on low-latency
- **Premium:**: No Tier Access, focused on low-latency
- **Live migration of storage redundancy**: LRS to ZRS Only

## Storage Tiers

- Hot / Cool / Cold (online tiers)
- Archive (offline tiers)
  - Up to 15 hours to transition to online tiers.
  - No Zone-related redudancy support, designed for Archive, not HA
- Default tier: Hot / Cool only
- File Share: Hot and Cool only
- Tier Durations: 30, 90, 180 days for cool, cold, archive

## Blob Features (except V1)

- **Tier Access:** Enables cost savings by managing data storage in different tiers (hot, cool, cold, archive)
- **Lifecycle Management:** Facilitates moving objects between tiers and setting retention periods for data
- **Access Policies:** Allows setting up to 5 access policies to limit SAS
- **Immutable Policies:** Supports configuring up to 2 immutable policies to prevent data modification or deletion
- **Object Replication:** Provides options to replicate objects to specific region GRS does not allow choosing the replication region
- **Scoped Encryption:** Allows using different encryption keys for specific containers

## Encryption

- Blob: support file-level encryption
- File Share: Data at rest encryption only
- Customer-provided key:
  - Blob: Supported
  - Data Lake: Not Support

## IAM Role Conditions Support

- Containers
- Queues

## Storage Type Comparision

- Page blob: Block-level backup, DB, OS disk
- Block blob: Large objects (e.g., video), up to 8TB
- Append blob: Logs
- File share: Legacy (SMB-like), up to 5TB/100TB with large file share feature

## Shared Access Signature (SAS)

A SAS is a secure way to share our Azure Storage resources without compromising the account keys.

It provides temporary, fine-grained access control to storage resources.

### Resource Levels

It can be applied to different resource levess:
- Service: blob, file, queue, table
- Container: e.g., blob container
- Object: e.g., specific blob

### Permissions

And can grant various permission, including:
- READ: Read data
- WRITE: Write data
- DELETE: Delete objects
- CREATE: Create new objects
- LIST: List objects in a container
- ADD: Add messages to a queue

### Usage Examples

| Operation                  | SAS Permissions               | Azure CLI Command                                                                                               |
|----------------------------|----------------------------------------|-------------------------------------------------------------------------------------------------------|
| Download Blob          | Resource Type: Object <br> Permissions: Read | ```az storage blob download --account-name "$account" --container-name "$container" --sas-token "$token" --name "$file"``` |
| List Blobs in Container| Resource Type: Containe <br> Permissions: List | ```az storage blob list --account-name "$account" --container-name "$container" --sas-token "$token"``` |
| List Containers        | Resource Type: Servic <br> Permissions: List | ```az storage container list --account-name "$account" --sas-token "$token"``` |

## Access levels

It offers two main access levels:
- **Public**: Allows anonymous read access for containers and blobs
- **Private**: Rquires authentication for all access

## Access Policies

As mentioned, SAS provides a way to grant limited access to objects in our storage account to others, without exposing the account keys.

Notice that:
- SAS is signed using the account key.
- If compromised, a SAS cannot be directly revoked.
- To handle the compromised SAS, we can
  - Rotate the account key (affects all SAS tokens signed with it).
  - Use an access policy to further restrict the SAS.

That's why Access Policy is needed. It
- Provides an additional layer of control over SAS tokens.
- Can be used to modify or revoke permissions without changing the account key.

Each container can have up to 5 access policies defined.

## Immutable Blob Storage Policy

Immutable storage can helps us to store business-critical data objects in a WORM (Write Once, Read Many) state. 

There are two types of immutability policies:

### Legal Hold

- Allows creating and reading blobs.
- Prevents modification or deletion of blobs.
- Useful for short-term scenarios or legal investigations.
- Can be removed when no longer needed.

### Time-Based Retention

- Allows WORM within a specified period.
- After the retention period, objects can be deleted but not modified.
- Retention period can be extended but not shortened.
- Useful for regulatory compliance.

## Lifecycle Management

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

## IAM with Conditions

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

## Encryption

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

## Network ACL

### Configuration Example

- **Bypass:** AzureServices
- **Virtual Network Rules:** None configured
- **IP Rules:** None configured
- **Default Action:** Allow

This configuration indeed allows all access by default. The "AzureServices" bypass allows trusted Microsoft services to access the storage account even if network rules are in place.

When no virtual network or IP rules are specified, and the default action is set to "Allow", public IP access is permitted from any network.

### Network Modes

1. **Enable Public Access:** 
   - **Default Action:** Allow
   
2. **Enable Access from Selected Virtual Networks and IP Addresses:**
   - **Firewall (IP Rules)**: Allows us to specify individual IP addresses or CIDR ranges that can access the storage account.
   - **Virtual Network (Virtual Network Rules)**: Allows us to specify Azure Virtual Networks that can access the storage account.
   - **Exceptions (Bypass)**: Allows us to bypass the firewall for certain Azure services.

3. **Disable Public Access:**
   - **Default Action:** Disabled
   - This is the most restrictive setting, blocking all public access. Access is only possible through specified virtual networks or IP addresses.

## Miscellaneous

1. Azure Backup / Site Recovery
- When using Azure Backup for VMs or Azure Site Recovery with a storage account that has restricted public access, we need to enable the "Allow trusted Microsoft services to access this storage account" option.

4. Event hub Capture:
- Event Hub capture does not support premium storage accounts.
- It works with Data Lake Storage Gen2 or standard blob storage accounts.
- The data format used for Event Hub capture is Avro.

5. File Share
- When using SMB protocol with Azure File Share and authenticating with AD, Azure AD DS can be utilized.
