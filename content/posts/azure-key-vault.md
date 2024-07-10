---
title: Azure Key Vault
date: 2024-07-10
tags:
  - azure
  - key vault
---

## Access control

Azure Key Vault offers two main methods for controlling access:

1. Access Policies (legacy)
2. Role-Based Access Control (RBAC) - recommended

### Access Policies

- Applied at the Key Vault level
- Permissions granted affect all keys within the vault
- Less granular control compared to RBAC

### RBAC

- Can be set at either the Key Vault or Object level
- Allows for more granular permissions (e.g., granting access to Key1 but not Key2)
- Recommended over Access Policies for better security and management

## Enabling Key Vault for Specific Services

To use Key Vault with certain Azure services, we need to enable specific permissions:

1. For Azure Disk Encryption:
   - Navigate to the Key Vault's Access Policies
   - Under "Enable Access to", select "Azure Disk Encryption for volume encryption"

2. For ARM Template deployments:
   - Enable "Access Azure Resource Manager for template deployment"

## Integrate with App Serivce 

To integrate Key Vault with an App Service:
- Create a Managed Identity for App.
- Configure IAM settings in Key Vault. (eg: Key Vault Secrets User)
- Add Application Setting to reference the key Vault Secret

```text
DB_PASSWORD=@Microsoft.KeyVault(SecretUri=https://mykeyvault.vault.azure.net/secrets/DB_PASSWORD/)
```

- In the Application, we can access the secret as environment variable
```php
<?php
// Connect to database
$servername = "your_server_name";
$username = "your_username";
$password = getenv('DB_PASSWORD');
$dbname = "your_database_name";

// Create connection
$conn = new mysqli($servername, $username, $password, $dbname);

// Check connection
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
echo "Connected successfully";
?>
```

## Failover

It supports failover within a region and across regions. 

However,

- During regional failover, the key vault becomes read-only.
- DELETE operations cannot be performed during this time.

The read-only state ensures data consistency across regions during the failover.

## Backup and Restore

Limitations:

- Backups can only be restored within the same geography.
- For example, a backup from UK West can be restored to UK East, but not to US East.
- Backups can only be restored within the same Azure subscription.

- Backups are encrypted and can only be decrypted within Azure.
- There is a time limit for restoring backups
- Keys, certificates, and secrets need to be backed up individually. There is no single operation to back up the entire key vault at once.
