---
title: Azure Powershell
date: 2024-06-26
tags:
  - azure
  - cli
---

### Naming Conventions

Prefix:
- `Azure`: Older prefix for Azure PowerShell cmdlets.
- `Rm`: Resource Manager (older prefix).
- `Az`: Newer prefix for Azure PowerShell cmdlets.

Actions:
- `Get-`: Returns an object. e.g. `Get-AzVM` Retrieves details of a virtual machine.
- `Set-`: Updates the property of an object. e.g. `Set-AzVMSize` Changes the size of a virtual machine.
- `Update-`: Updates a resource with the object. e.g. `Update-AzVM` Updates a virtual machine with new configuration settings.
- `New-`: Creates a resource. e.g. `New-AzVM` Creates a new virtual machine.

## Azure AD

### Add AD User

New-AzureADUser
```powershell
$userPrincipalName = "newuser@yourtenant.onmicrosoft.com"
$displayName = "New User"
$passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
$passwordProfile.Password = "P@ssw0rd1234"

New-AzureADUser -DisplayName $displayName -PasswordProfile $passwordProfile -UserPrincipalName $userPrincipalName -AccountEnabled $true -MailNickname "newuser"
```

New-MgUser
```powershell
$userPrincipalName = "newuser@yourtenant.onmicrosoft.com"
$displayName = "New User"
$password = "P@ssw0rd1234"

New-MgUser -UserPrincipalName $userPrincipalName -DisplayName $displayName -PasswordProfile @{ Password = $password } -MailNickname "newuser" -AccountEnabled $true
```

### Invite AD Guest User

New-AzureADMSInvitation
```powershell
$invitedUserEmailAddress = "guestuser@example.com"
$inviteRedirectUrl = "https://myapplication.com/welcome"

New-AzureADMSInvitation -InvitedUserEmailAddress $invitedUserEmailAddress -InviteRedirectUrl $inviteRedirectUrl -SendInvitationMessage $true
```

New-MgInvitation
```powershell
$invitedUserEmailAddress = "guestuser@example.com"
$inviteRedirectUrl = "https://myapplication.com/welcome"

New-MgInvitation -InvitedUserEmailAddress $invitedUserEmailAddress -InviteRedirectUrl $inviteRedirectUrl -SendInvitationMessage $true
```

### Get Role Definition

- `Get-AzRoleDefinition`: Get Azure role definition.

### AD Sync

- `Start-ADSyncSyncCycle -PolicyType Initial`: Full sync of AD Connect (slow).
- `Start-ADSyncSyncCycle -PolicyType Delta`: Delta sync of AD Connect (quick).

## VM

### Create VM

- `New-AzureRmVM`
- `New-AzVM`
- `New-AzureVM`

### Upload Virtual Disk to Storage Account

```powershell
Add-AzVhd -ResourceGroupName "ResourceGroupName" -Destination "https://storageaccount.blob.core.windows.net/containername/diskname.vhd" -LocalFilePath "C:\Path\To\My\Local.vhd"
```

### Create VM from VHD

1. Create Image from VHD.
2. Create VM with VM Config.

Commands:
- `New-AzImageConfig`
- `Set-AzImageOsDisk`
- `Add-AzImageDataDisk`
- `New-AzImage`
- `New-AzureVMConfig`
- `New-AzureVM`

```powershell
$resourceGroupName = "MyResourceGroup"
$location = "EastUS"
$imageName = "MyCustomImage"
$vmName = "MyNewVM"
$vmSize = "Standard_DS1_v2"
$osDiskUri = "https://mystorageaccount.blob.core.windows.net/vhds/myosdisk.vhd"
$dataDiskUri = "https://mystorageaccount.blob.core.windows.net/vhds/mydatadisk.vhd"
$storageAccount = "mystorageaccount"
$containerName = "vhds"
$osDiskName = "$vmName-osdisk.vhd"

$imageConfig = New-AzImageConfig -Location $location
$imageConfig = Set-AzImageOsDisk -Image $imageConfig -OsType Windows -OsState Generalized -BlobUri $osDiskUri
$imageConfig = Add-AzImageDataDisk -Image $imageConfig -Lun 0 -BlobUri $dataDiskUri
$image = New-AzImage -ResourceGroupName $resourceGroupName -ImageName $imageName -Image $imageConfig

$vmConfig = New-AzureVMConfig -VMName $vmName -VMSize $vmSize
$vmConfig = Set-AzVMOSDisk -VM $vmConfig -Name $osDiskName -CreateOption FromImage -SourceImageUri $image.StorageProfile.OsDisk.Image.Uri -StorageAccountName $storageAccount -ContainerName $containerName

New-AzVM -ResourceGroupName $resourceGroupName -Location $location -VM $vmConfig
```

### Update VM

1. Get the VM object.
2. Update the config (IP/subnet).
3. Call Update command to apply changes.

Commands:
- `Get-AzureVM`
- `Set-AzureStaticVNetIP`
- `Set-AzureSubnet`
- `Update-AzureVM`

```powershell
$resourceGroupName = "MyResourceGroup"
$vmName = "MyExistingVM"
$newStaticIP = "10.0.0.10"
$newSubnetName = "MyNewSubnet"

$vm = Get-AzureVM -ResourceGroupName $resourceGroupName -Name $vmName
$vm = Set-AzureStaticVNetIP -VM $vm -IPAddress $newStaticIP
$vm = Set-AzureSubnet -VM $vm -SubnetName $newSubnetName
Update-AzureVM -ResourceGroupName $resourceGroupName -VM $vm
```

## AKS

### Set AKS Config

```powershell
Set-AzAksCluster
```

### Build and Push Image

```powershell
az acr build --registry <registry-name> --image <image-name>:<tag> <local-context-path>
```

### Create AKS

```powershell
az aks create
```

### Update Node Pool Config

```powershell
az aks nodepool update
```

## ARM Template

### Deploy ARM Template

- At Management Group: `New-AzManagementGroupDeployment`
- At Subscription: `New-AzSubscriptionDeployment`
- At Resource Group: `New-AzResourceGroupDeployment`

```powershell
$templateFile = "path/to/template.json"
$parametersFile = "path/to/parameters.json"

$managementGroupName = "MyManagementGroup"
New-AzManagementGroupDeployment -ManagementGroupId $managementGroupName -TemplateFile $templateFile -TemplateParameterFile $parametersFile

$subscriptionId = "MySubscriptionId"
New-AzSubscriptionDeployment -SubscriptionId $subscriptionId -TemplateFile $templateFile -TemplateParameterFile $parametersFile

$resourceGroupName = "MyResourceGroup"
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateFile $templateFile -TemplateParameterFile $parametersFile
```

## azcopy

### Create Container

```powershell
azcopy make "https://<storage-account-name>.blob.core.windows.net/<container-name>"
```

### Copy Folder Recursively

```powershell
azcopy copy "<local-folder-path>" "https://<storage-account-name>.blob.core.windows.net/<container-name>" --recursive
```

### Sync 

like rsync, transfer the delta only

```powershell
azcopy sync "<local-folder-path>" "https://<storage-account-name>.blob.core.windows.net/<container-name>"
```

## Others

### Accept Marketplace Terms

```powershell
$publisher = "publisher-name"
$offer = "offer-name"
$plan = "plan-name"

Set-AzMarketplaceTerms -Publisher $publisher -Product $offer -Name $plan -Terms $marketplaceTerms -Accept
```