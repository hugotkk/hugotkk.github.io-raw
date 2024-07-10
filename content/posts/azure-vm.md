---
title: Azure VM
date: 2024-07-09
tags:
  - azure
  - vm
---

## Instance Types

- D series: General Purpose; S for SSD
- M / E series: Memory optimized
- F series: Compute optimized
- H series: HPC
- L series: Disk optimized
- N series: GPU (NVIDIA)

## DNS Formats

- internal: `<vm-name>.internal.cloudapp.azure.com`
- public: `<vm-name>.<region>.cloudapp.azure.com`

## SetupComplete.cmd

- Runs once during the Windows out-of-box experience (OOBE) setup
- Used for one-time configuration tasks after Windows installation
- Similar to cloud-init in Linux environments
- Location: %WINDIR%\Setup\Scripts\SetupComplete.cmd
- Runs with SYSTEM privileges

GPO - Logon Scripts: Runs every time a user logs into the computer (User Context)
GPO - Start Scripts: Runs every time the computer starts (SYSTEM privileges)

When deciding between these options:
- If the task needs to be performed only once after installation, use SetupComplete.cmd
- For recurring tasks or configurations that need to be enforced regularly, use GPO scripts
- SetupComplete.cmd is useful in imaging and deployment situation where we want to perform actions immediately after installation

## Desired State Configuration

It is similar to Chef for Configuration Management.

We can enable DSC at "Extension" section

When setup DSC, VM need to be in running state 

For example,

It can be used for configuration management (install Nginx) on Windows

Save InstallNginx.ps1
```
Configuration InstallNginx {
    Node "localhost" {
        Package Nginx {
            Ensure = "Present"
            Name = "nginx"
            Source = "https://nginx.org/packages/windows/"
        }
    }
}
```

Update InstallNginx.ps1.zip to storage account

Add DSC Extension to run the script

```
Set-AzVMDscExtension -ResourceGroupName "MyResourceGroup" `
                     -VMName "MyVM" `
                     -ArchiveBlobName "InstallNginx.ps1.zip" `
                     -ArchiveStorageAccountName "mystorageaccount" `
                     -ConfigurationName "InstallNginx" `
                     -Version "2.76"
```

Although we can Azure Custom Script Extension to install Nginx as well, DSC is more suitable for maintaining a desired configuration state, while Custom Script Extension is better for one-time script execution.

## Redeploy

Any data stored on the temporary disk will be lost when the VM is redeployed, resized / restarted.

On Windows, the temporary disk is assigned to drive `D:\``
On Linux, it's located at `/dev/sdb1`

## Import VHDx to Azure

- Convert the VHDx to VHD before importing it
- Ensure the VHD is in fixed-size format, not dynamically expanding.

Convert VDHX to VHD
```
Convert-VHD -Path "C:\Path\To\Your\DiskFile.vhdx" -DestinationPath "C:\Path\To\Output\DiskFile.vhd" -VHDType Fixed
```

Verify the conversion
```
Get-VHD -Path "C:\Path\To\Output\DiskFile.vhd"
```

Import to Azure
```
Add-AzVhd -ResourceGroupName "MyResourceGroup" -LocalFilePath "C:\Path\To\Output\DiskFile.vhd" -Destination "https://mystorageaccount.blob.core.windows.net/vhds/DiskFile.vhd"
```

## Disk Operations

Data Disks:
- Can be detached from a running VM without stopping it.

OS disk
- Requires the VM to be stopped and deallocated to make changes.

## Host Groups

- Is a collection of dedicated hosts
- All hosts in a group must be of the same size (e.g., DSv3, ESv3)
- Located within a single Availability Zone in a region

Dedicated Hosts:
- Physical hosts in Azure datacenters dedicated to us (rent the whole physical server!)

## ScaleSet

There are two mode for managing VMSS:
- Orchestration: VMs are managed automatically by Azure
- VM: we need to manually add or remove VM instances

### Scaling Mechanisms

It supports both manual and automatic scaling based on defined rules and metrics. 

There are 2 key concepts about the scaling:
- Duration: The period over which the metric is observed.
- Cooldown: The period to wait after a scale-out or scale-in action before another scaling action can occur

For Example, we have scaling rule:
- If CPU > 70% for 5 minutes, scale up by 1 instance.
- Minimum instances: 1
- Maximum instances: 3
- Default instances: 1
- Cooldown period: 5 minutes

CPU > 70% lasted for 15 minutes

- 5min: Scale up by 1 instance (total: 2)
- 5-10min: 5 min cooldown period
- 10min: Scale up by 1 instance (total: 3)
- 10-15min: 5 min cooldown period
- 15min: No scale up happened as Max instances is reach

## Host Groups

We can create VMSS using host group

- The scale set can only exist in a single AZ. As all hosts in a host group are in the same AZ
- To achieve HA across multiple AZs, we need to create multiple scale sets
- Each scale set would use a host group in a different AZ
- We can use Azure Load Balancer or an Application Gateway to distribute traffic across these multiple scale sets

## Planned Upgrade
- No more than 20% of the scale set is upgrading at any time.
- For a scale set with 10 VMs, a maximum of 2 VMs are upgraded at a time, leaving 8 VMs operational.

## Placement Group

A placement group must be in the same region (location) as the scale set it's associated with.

## Availability Set

Availability sets:
- Provide high availability within a single data center
- Use fault domains and update domains to distribute VMs across different physical hardware
- Protect against hardware failures and planned maintenance events

Availability zones:
- Provide HA across multiple data centers within a region
- Each zone is a separate physical location with independent power, cooling, and networking
- Protect against data center-level failures.

### Fault and Update Domains

- Fault Domain is for unplanned outage
- Update Domain is for planned outage

Limitation:
- When Fault Domains is set to 1, Update Domains can only be 1
- If Fault Domains is > 2, Update Domains can be up to 20

For example:

- Fault Domain: 2
- Update Domain: 10
- Total VMs: 11

| Fault Domain | VMs |
|--------------|-----|
| 1            | 2   |
| 2            | 1   |
| 3            | 1   |
| 4            | 1   |
| 5            | 1   |
| 6            | 1   |
| 7            | 1   |
| 8            | 1   |
| 9            | 1   |
| 10           | 1   |

Max no of unavailable VMs during planned maintenance is 2 VMs (as 1: 2 VMs)

| Fault Domain | VMs Distribution |
|--------------|------------------|
| 1            | 6 VMs            |
| 2            | 5 VMs            |

Max no of unavailable VMs during unplanned maintenance is 6 VMs (as 1: 6 VMs)

### Resizing Availability Set
- Stop and deallocate all VMs in the availability set
- Resize the availability set
- Restart the VMs