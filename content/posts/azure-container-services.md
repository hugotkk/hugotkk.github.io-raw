---
title: Azure Container Services
date: 2024-07-09
tags:
  - azure
  - aks
  - acr
  - container instance
  - container app
---

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

Limitations:
- No support for DaemonSets or init containers
- No IPv6 support
- Only compatible with Azure CNI

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


## ACR

Geo-replication is an exclusive feature available in the premium SKU only.

### Integrate with AKS

```
az aks update -g myResourceGroup -n myAKSCluster --enable-managed-identity
```

```
az aks update --name myAKSCluster --resource-group myResourceGroup --attach-acr <acr-name>
```

With this command, AKS will configures the AcrPull role for the managed identity, allowing the AKS cluster to pull images from the specified ACR


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

## Container Instance

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