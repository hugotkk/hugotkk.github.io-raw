---
title: Azure ARM Template
date: 2024-07-06
tags:
  - azure
  - arm template
  - blueprint
---

We can view resources deployed via ARM templates at:
- Resource Group > Deployments

When using `count`, the `copyIndex()` function starts counting from 0.

```json
"resources": [
    {
        "type": "Microsoft.Storage/storageAccounts",
        "name": "[concat('storage', copyIndex())]",
        "apiVersion": "2022-01-01",
        "location": "[resourceGroup().location]",
        "count": 3
    }
]
```

This example will deploy 3 storage accounts named storage0, storage1, and storage2.

We can use `resourceId()` to reference the existing resource. The following example show how to reference a resource named VM1Nic1

```json
{
    "name": "VM1",
    "type": "Microsoft.Compute/virtualMachines",
    "apiVersion": "2022-01-01",
    "location": "[parameters('vmLocation')]",
    "properties": {
        "networkProfile": {
            "networkInterfaces": [
                {
                    "id": "[resourceId('Microsoft.Network/networkInterfaces', 'VM1Nic1')]"
                }
            ]
        }
    }
}
```

`ImageReference` is used to choose the OS image when creating VM.

To find the values for `publisher`, `offer`, `sku`, and `version`, I usually simply pretend to create a VM in Azure Portal and look up these values during the setup process.

```json
{
    "name": "VM1",
    "type": "Microsoft.Compute/virtualMachines",
    "apiVersion": "2022-01-01",
    "location": "[parameters('vmLocation')]",
    "properties": {
        "storageProfile": {
            "imageReference": {
                "publisher": "MicrosoftWindowsServer",
                "offer": "WindowsServer",
                "sku": "2019-Datacenter",
                "version": "latest"
            }
        }
    }
}
```

Differences between variables and parameteers:
- **Parameters:** Customizable. We can provide these values when we use the template or script.
- **Variables:** These values are calculated based on the parameters or other logic.

This example show that we can use parameters to change which vmSize use to deploy. and the storage account name is concat from storage + resource group id
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmSize": {
      "type": "string",
      "defaultValue": "Standard_DS1_v2",
      "allowedValues": [
        "Standard_DS1_v2",
        "Standard_DS2_v2"
      ],
      "metadata": {
        "description": "The size of the virtual machine."
      }
    }
  },
  "variables": {
    "storageAccountName": "[concat('storage', uniqueString(resourceGroup().id))]"
  },
  "resources": [
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2020-06-01",
      "name": "[concat('vm', uniqueString(resourceGroup().id))]",
      "location": "[resourceGroup().location]",
      "properties": {
        "hardwareProfile": {
          "vmSize": "[parameters('vmSize')]"
        },
        "storageProfile": {
          "osDisk": {
            "createOption": "FromImage",
            "name": "[variables('storageAccountName')]"
          }
        }
      }
    }
  ]
}
```

## Powershell

We can deploy an ARM template at different levels:

- Management Group: Use `New-AzManagementGroupDeployment`.
- Subscription: Use `New-AzSubscriptionDeployment`.
- Resource Group: Use `New-AzResourceGroupDeployment`.

When deploying with `New-AzResourceGroupDeployment`, there are two modes:

- **Complete:** Deletes resources in the resource group that are not specified in the template.
- **Incremental:** Retains existing resources in the resource group that are not specified in the template.

Example usage:
```powershell
New-AzResourceGroupDeployment -TemplateUri xxx -TemplateParameterFile params.json -ResourceGroupName RG1 -Mode Complete
```

Regarding `New-AzSubscriptionDeployment`, the `-Location` parameter only affects where the ARM template itself is stored, not the location of the resources being deployed.

```json
$templateFile = "path/to/your/template.json"
$parameterFile = "path/to/your/parameters.json"

New-AzSubscriptionDeployment `
  -Name "ExampleDeployment" `
  -Location "eastus" `
  -TemplateFile $templateFile `
  -TemplateParameterFile $parameterFile
```

To deploy a resource within a subscription, we cannot do it directly. We need to wrap it within a deployment resource.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.Resources/resourceGroups",
      "apiVersion": "2021-04-01",
      "location": "[parameters('location')]",
      "name": "[parameters('resourceGroupName')]",
      "properties": {}
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2021-04-01",
      "name": "storageAccountDeployment",
      "resourceGroup": "[parameters('resourceGroupName')]",
      "dependsOn": [
        "[resourceId('Microsoft.Resources/resourceGroups', parameters('resourceGroupName'))]"
      ],
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "resources": [
            {
              "type": "Microsoft.Storage/storageAccounts",
              "apiVersion": "2021-04-01",
              "name": "[parameters('storageAccountName')]",
              "location": "[parameters('location')]",
              "sku": {
                "name": "Standard_LRS"
              },
              "kind": "StorageV2",
              "properties": {}
            }
          ]
        }
      }
    }
  ],
  "parameters": {
    "location": {
      "type": "string",
      "defaultValue": "eastus",
      "metadata": {
        "description": "Location for the resources."
      }
    },
    "resourceGroupName": {
      "type": "string",
      "metadata": {
        "description": "Name of the resource group to create."
      }
    },
    "storageAccountName": {
      "type": "string",
      "metadata": {
        "description": "Name of the storage account to create."
      }
    }
  }
}
```


## Template Library

To create template:

1. Navigate to an existing VM in the Azure portal
2. Select "Export template" from the left menu
3. Review the generated template
4. Click "Add to library" to save it as a Template Spec

To use the Template:

1. Go to the desired Resource Group
2. Click "New" or "Add"
3. Search for and select "Template deployment"
4. In "Template Source", select "Template spec"
5. Choose the saved template
7. Fill in any required parameters
8. Review and create

OR

1. Navigate to "Template specs" in the Azure portal
2. Select the specific template that want to deploy
3. Click "Deploy"
4. Choose the target subscription and resource group
5. Fill in any required parameters
6. Review and deploy

## Virtual Machine Extensions

To domain join AD in a VM, we need to deploy Virtual Machine Extensions (`Microsoft.Compute/virtualMachines/extensions`) with the extension type `JsonADDomainExtension` in Template.

In the **settings**, provide 
- domain name
- username for domain joining
- OUpath

In the **protectedSettings**, proivde
- domain join account password

``` json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string"
    },
    "domainName": {
      "type": "string"
    },
    "domainUsername": {
      "type": "string"
    },
    "domainPassword": {
      "type": "securestring"
    },
    "ouPath": {
      "type": "string",
      "defaultValue": ""
    }
  },
  "resources": [
    {
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "apiVersion": "2021-03-01",
      "name": "[concat(parameters('vmName'), '/JsonADDomainExtension')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "publisher": "Microsoft.Compute",
        "type": "JsonADDomainExtension",
        "typeHandlerVersion": "1.3",
        "autoUpgradeMinorVersion": true,
        "settings": {
          "Name": "[parameters('domainName')]",
          "User": "[parameters('domainUsername')]",
          "Restart": "true",
          "Options": "3",
          "OUPath": "[parameters('ouPath')]"
        },
        "protectedSettings": {
          "Password": "[parameters('domainPassword')]"
        }
      }
    }
  ]
}
```

## Blueprint

Blueprints in Azure are similar to CloudFormation in AWS. They:
- Incorporate role assignments, policy assignments, and ARM templates.
- Handle the lifecycle of deployed resources.
- Support versioning and updates.

ARM Templates, on the other hand:
- Are declarative JSON files that define Azure infrastructure.
- Specify which resources to deploy but do not manage their ongoing state.

Blueprints can be stored at either the management group or subscription level.

To apply a Blueprint, it must be assigned to a subscription.
