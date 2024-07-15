---
title: Azure App Service
date: 2024-07-09
tags:
  - azure
  - app service
---

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

## Tiers

### Free
- Shared infrastructure
- Limited compute resources
- No custom domain or SSL support

### Basic, Standard, Premium (Dedicated)
- Dedicated VMs
- Support for VNet integration (in Standard and above)
- Auto-scaling (in Standard and above)

### Isolated (App Service Environment)
- Fully isolated and dedicated environment
- Enhanced security features
- Highest scale and performance options

## Runtime Support

Both Windows and Linux
- .NET Core 3.0 and later
- PHP

Linux only
- Ruby

Windows only
- ASP.NET (traditional .NET Framework)

## HA

For HA or DR scenarios, we can deploy App Services across multiple regions:

- Create identical App Services in separate regions.
- One region serves as the active site while the other acts as standby.
- Azure Front Door can be used to route traffic between these regions.

## Price Tiers

- **Standard and Premium Tiers**:
  - **Scaling:**
    - Horizontal: Increase the no of VM instances
    - Vertical: Upgrade to a higher instance type
  - Slot Deployment: Available for managing multiple deployment slots
- **Isolated Tier**: 
  - Run app on a dedicated VNet, providing network isolation and improved security.
  - Can integrate with App Service Environment (ASE) - which run app on a dedicated host

## Key Vault

To use Azure Key Vault in an App Service
- Create Managed User Identity in App Service
- Config IAM in Key Vault
- Reference Key Vault secrets in App Service configuration

## Logging

- App Service Logs: Web Server Logging
- Application Insights Profile: Performance Traces

## Backup

App service backups are stored in a Storage Account.

We can create `_backup.filter` to exclude specific folders from the backup.

```yaml
-*.avi
-*.mp4
-/wwwroot/large_files
+/wwwroot/important_folder
```