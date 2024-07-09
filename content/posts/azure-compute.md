---
title: Azure Computes
date: 2024-07-09
tags:
  - azure
  - aks
  - vm
  - scaleset
  - app service
  - container instance
---

## VM

### Instance Type

- D series: General Purpose; S for SSD
- M / E series: Memory optimized
- F series: Compute optimized
- H series: HPC
- L series: Disk optimized
- N series: GPU (NVIDIA)

## Disk Type

### SSD
- Standard SSD: General Purpose
- Premium SSD: Support high IOPS
- Ultra Disk: High-performance option (Overskill for most scenarios)

### HDD
- Standard HDD

### DNS Format

internal: `<vm-name>.internal.cloudapp.azure.com`
public: `<vm-name>.<region>.cloudapp.azure.com`

### SetupComplete.cmd

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

### Desired State Configuration

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

### Redeploy

Any data stored on the temporary disk will be lost when the VM is redeployed, resized / restarted.

On Windows, the temporary disk is assigned to drive `D:\``
On Linux, it's located at `/dev/sdb1`

### Import VHDx to Azure

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

### Disk Operation

Data Disks:
- Can be detached from a running VM without stopping it.

OS disk
- Requires the VM to be stopped and deallocated to make changes.

## Scale Set

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

### Planned Upgrade
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

### Resizing Availability Sets
- Stop and deallocate all VMs in the availability set
- Resize the availability set
- Restart the VMs

## App Service

The region of the App and App Service Plan should be matched.

App Service Plan defines the underlying infrastructure and resources for hosting applications. 

This includes the OS (Windows/Linux), Region, no of instances, instance type, and pricing tier.

App refers to the actual application or code that runs on the App Service Plan.

Multiple apps can share a single App Service Plan, which helps optimize resource utilization and costs.

| Service            | Supported Operating Systems |
|--------------------|-----------------------------|
| Container Instance | Linux, Windows              |
| Container App      | Linux                       |
| App Service        | Linux, Windows              |
| AKS                | Windows, Linux              |

### Runtime Support

Both Windows and Linux
- .NET Core 3.0 and later
- PHP

Linux only
- Ruby

Windows only
- ASP.NET (traditional .NET Framework)

### HA

For HA or DR scenarios, we can deploy App Services across multiple regions:

- Create identical App Services in separate regions.
- One region serves as the active site while the other acts as standby.
- Azure Front Door can be used to route traffic between these regions.

### Price Tiers

- **Standard and Premium Tiers**:
  - **Scaling:**
    - Horizontal: Increase the no of VM instances
    - Vertical: Upgrade to a higher instance type
  - Slot Deployment: Available for managing multiple deployment slots
- **Isolated Tier**: 
  - Run app on a dedicated VNet, providing network isolation and improved security.
  - Can integrate with App Service Environment (ASE) - which run app on a dedicated host

### Key Vault

To use Azure Key Vault in an App Service
- Create Managed User Identity in App Service
- Config IAM in Key Vault
- Reference Key Vault secrets in App Service configuration

### Logging

- App Service Logs: Web Server Logging
- Application Insights Profile: Performance Traces

### Backup

App service backups are stored in a Storage Account.

We can create `_backup.filter` to exclude specific folders from the backup.

```yaml
-*.avi
-*.mp4
-/wwwroot/large_files
+/wwwroot/important_folder
```

## Container App

- Smallest subnet is /23. (It requires minimum of 512 addresses)
- `restartPolicy: onFailure`: Ensures that the container will restart if it crashes.
- `ipAddress.type: public`: Makes the container publicly accessible.

```yaml
apiVersion: 2019-12-01
type: Microsoft.ContainerInstance/containerGroups
name: my-container-group
location: eastus
properties:
  containers:
  - name: my-container
    properties:
      image: nginx:latest
      ports:
      - port: 80
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
  restartPolicy: OnFailure
  ipAddress:
    type: Public
    ports:
    - protocol: TCP
      port: 80{
  "type": "Microsoft.ContainerInstance/containerGroups",
  "apiVersion": "2021-09-01",
  "name": "mycontainergroup",
  "location": "eastus",
  "properties": {
    "containers": [
      {
        "name": "mycontainer",
        "properties": {
          "image": "mcr.microsoft.com/azuredocs/aci-helloworld:latest",
          "ports": [
            {
              "port": 80
            }
          ],
          "resources": {
            "requests": {
              "cpu": 1,
              "memoryInGB": 1.5
            }
          }
        }
      }
    ],
    "restartPolicy": "OnFailure",
    "ipAddress": {
      "type": "Public",
      "ports": [
        {
          "protocol": "tcp",
          "port": 80
        }
      ]
    },
    "osType": "Linux"
  }
}
```

### Container Instance

### DNS name label

The DNS name label scope reuse is only available in the public networking.

There are three levels of DNS name label reuse:
- Tenant
- Subscription
- Resource Group

When we create a container instance, it generates a DNS name label for it.

This label, known as the DNS name label, can be reused if you recreate the container, meaning it will either retain the same DNS name label (reuse) or assign a new one.

For example,

The DNS name label `23098djd` in `hugo-23098djd.<region>.azurecontainer.io` is generated by Azure. 

If we delete and recreate the container with the same name (`hugo` in this case), Azure will reuse the same `23098djd` DNS name label. 

### Container Groups

Container Groups in ACI are similar to pods in Kubernetes. 

They are the top-level resource in ACI and can contain one or more containers that are scheduled on the same host machine.

Windows:
- Limited to single container per group

Linux:
- Support multiple contianers within single group

```json
{
  "name": "myContainerGroup",
  "type": "Microsoft.ContainerInstance/containerGroups",
  "apiVersion": "2023-05-01",
  "location": "[resourceGroup().location]",
  "properties": {
    "containers": [
      {
        "name": "container1",
        "properties": {
          "image": "mcr.microsoft.com/azuredocs/aci-helloworld:latest",
          "ports": [
            {
              "port": 80
            }
          ],
          "resources": {
            "requests": {
              "cpu": 1,
              "memoryInGB": 1.5
            }
          }
        }
      },
      {
        "name": "container2",
        "properties": {
          "image": "mcr.microsoft.com/azuredocs/aci-tutorial-sidecar",
          "resources": {
            "requests": {
              "cpu": 1,
              "memoryInGB": 1.5
            }
          }
        }
      }
    ],
    "osType": "Linux",
    "ipAddress": {
      "type": "Public",
      "ports": [
        {
          "protocol": "TCP",
          "port": 80
        }
      ]
    }
  }
}

```

## AKS

### Azure AD Authentication

Legacy Method:
- Create an app in EntraID manually
- Assign directory read role to the app
- Link the app with AKS

```bash
az aks update-credentials \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --reset-aad \
  --aad-client-app-id <client-id> \
  --aad-server-app-id <server-id> \
  --aad-server-app-secret <server-secret> \
  --aad-tenant-id <tenant-id>
```

Current Method:
- AKS service creates the necessary Azure AD application automatically
- We just need to enable the feature when creating or updating the cluster

```bash
az aks update --resource-group myResourceGroup --name myAKSCluster --enable-aad
```

### Networking

#### Node Networking

By default, AKS use Azure Managed Virtual Network and subnet for nodes.

We can use Bring Your Own Virtual Network (BYOVNET) to use our existing VNET for nodes.

"Enabling public IP per node" doesn't directly affect the cluster's internet access. 

The cluster can still access the internet even with this option disabled, unless we configure "Enable private cluster" to Yes.

#### Pod Networking

- Azure CNI
- Kubenet

Azure CNI:
- each pod is assigned an IP address that's individually accessible within the vnet
- useful when we need direct communication with pods using their IP addresses.
- Both the VNET and AKS cluster must be in the same location

kubenet:
- pods will form a internal network which use within the cluster only
- The VNET can be in any location, but it needs to be routable to the AKS cluster

Notice that **Windows node pools** and **Virtual Node** are only supported with **Azure CNI** networking.

### Network Policy

- **Calico**: Can be used with both kubenet and Azure CNI pod networking
- **Azure**: Must be used with Azure CNI pod networking

### Virtual Node

Virtual nodes in AKS use Azure Container Instances (ACI) as the underlying infrastructure, similar to how AWS Fargate works for EKS

Virtual nodes can be enabled as an add-on:
```
az aks enable-addons --addons virtual-node --name myAKSCluster --resource-group myResourceGroup
```

To deploy applications to virtual nodes, we need to use specific nodeSelector and tolerations in pod specification

```yaml
      nodeSelector:
        kubernetes.io/role: agent
        beta.kubernetes.io/os: linux
        type: virtual-kubelet
      tolerations:
      - key: virtual-kubelet.io/provider
        operator: Exists
      - key: azure.com/aci
        effect: NoSchedule
```

### Windows Container

#### Windows Node Pool

```
az aks nodepool add \
  --resource-group <resource-group-name> \
  --cluster-name <cluster-name> \
  --os-type Windows \
  --name <windows-nodepool-name> \
  --node-count <number-of-nodes> \
  --kubernetes-version <kubernetes-version> \
  --node-taints "os=windows:NoSchedule"
```

#### Manual-installed Virtual Node

Azure-managed virtual node add-on for AKS does not natively support Windows containers. 

However, there is a way to use Windows containers with virtual nodes in AKS, through a manual installation process. 

See https://github.com/virtual-kubelet/azure-aci/blob/master/docs/windows-virtual-node.md

### Integrate ACR with AKS

```
az aks update -g myResourceGroup -n myAKSCluster --enable-managed-identity
```

```
az aks update --name myAKSCluster --resource-group myResourceGroup --attach-acr <acr-name>
```

With this command, AKS will configures the AcrPull role for the managed identity, allowing the AKS cluster to pull images from the specified ACR
