---
title: Azure Storage Tools
date: 2024-06-26
tags:
  - azure
  - azcopy
  - import/export
  - file sync
  - azure file explorer
---

## Summary

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

AzCopy is a command-line tool designed for copying data to and from Azure Storage, including both file and blob storage. 

It supports:

Authentication:
- AD
- SAS

Platforms:
- Windows
- Linux
- macOS
- No Android / iPhone

## Azure File Explorer

Capability:
- Cannot create storage accounts.
- Can change blob types during file uploads, choosing from append, block, or page.
- Can create file shares, containers, queues, and tables.
- Can also add data to tables and queues.