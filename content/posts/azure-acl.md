---
title: Azure ACL
date: 2024-06-29
tags:
  - azure
  - role
---

## IAM Roles

Access rights to Azure resources:

- Owner: Full access, including assigning roles and policies.
- User Access Administrator: Assign roles/policies.
- Contributor: Create and manage (read and write) resources.
- Reader: View resources (read-only).

Co-admin (legacy):

- A co-admin can only be added at the subscription level.
- A co-admin cannot be assigned to a management group, resource group or resource.
- A co-admin has full access to all resources within the subscription.

### Logic Apps

- Logic App Standard Developer: Develop and edit workflows/connections.
- Logic App Operator: Operate and manage workflows (enable/disable, resubmit, create connections).

### Storage Account

To view files in a storage account, we need the Reader role.

The Contributor role allows us to update storage account settings and create containers.

However, the Reader and Contributor roles do not include DataActions, so they cannot perform operations on the files level. (such as upload / download / delete)

To read/write files or blobs, we need the following `Data` roles:

- Storage Blob Data: For blob.
- Storage File Data: For file share.
- Storage File Data SMB: For file share in SMB.

| Role Name                                      | Read Access | Write Access | Delete Access | Modify Windows ACLs | Override Existing Windows ACLs |
|------------------------------------------------|-------------|--------------|---------------|---------------------|---------------------------------|
| Storage File Data Privileged Contributor       | Yes         | Yes          | Yes           | Yes                 | Yes                             |
| Storage File Data Privileged Reader            | Yes         | No           | No            | No                  | Yes                             |
| Storage File Data SMB Share Contributor        | Yes         | Yes          | Yes           | No                  | No                              |
| Storage File Data SMB Share Elevated Contributor | Yes       | Yes          | Yes           | Yes                 | No                              |
| Storage File Data SMB Share Reader             | Yes         | No           | No            | No                  | No                              |

By default, File Share doesn't work with Azure AD. You use an account key to connect to SMB.

To use Azure AD for authentication:
- Turn on Azure AD integration
- Choose "Enable permission for all authenticated users and groups"
- Pic a default role for the share

When using SMB, two types of permissions apply:
- Azure RBAC
- Windows ACL

The stricter permission wins. For example:
- An SMB Share Contributor can read and write files
- But Windows ACL can override this
- An Elevated Contributor can change Windows ACL (except ownership)
- Some files might still be inaccessible even for a Contributor

When backing disk to a storage account, we need to enable the "Allow trusted Microsoft services to access this storage account" option.

### Others

- Monitor Contributor/Network Contributor: Need to enable traffic analytics.
- Virtual Machine User Login: Login to VMs.
- Disk Snapshot Contributor: Manage disk snapshots.
- Website Contributor: Deploy content in web apps.
- DevTest Lab User: Use DevTest Labs.
- Resource Policy Contributor: Create and assign Azure policies.

## Entra Roles

Entra roles focus on managing Azure AD and don't have direct rights to manage Azure resources.

But we can temporarily elevate a Global Administrator to manage the resources:

1. Sign in as a Global Administrator.
2. Navigate to Azure Active Directory > Properties > Manage.
3. Enable "Access management for Azure resources"

Notice that:
- This elevation should be temporary, only for making necessary changes to Azure Resources.
- It's a backdoor option, meant for use only if access to resources is lost.
- This elevation applies only to the current signed-in user, not to all Global Administrators.

Key Roles in Entra:

1. **Global Administrator**: Has full control over the entire tenant, including the ability to manage all aspects of users and services.

2. **User Administrator**: Can create and manage users, but lacks the ability to control administrative accounts.

3. **Security Administrator**: Manages security settings such as editing and viewing security policies, and handling alerts and recommendations.

4. **Cloud Device Administrator**: Manages devices within Azure AD, including enabling, disabling, and deleting devices, as well as viewing BitLocker keys. They cannot add new devices to the network.

5. **Intune Device Administrator**: Manages devices specifically within Microsoft Intune, focusing on device and application management such as Mobile Device Management (MDM) and Mobile Application Management (MAM).

## Custom Roles

Custom Roles allow us to define specific permissions that aren't covered by built-in roles.

Components of a custom role include:

- Scopes
- Permissions:
  - `Actions`
  - `NotActions`
  - `DataActions`
  - `NotDataActions`

A custom role can be assigned multiple scopes that define its applicability.

The scope and permissions of a custom role can be modified even after its creation.

### Scopes

- Subscription: `/subscription/<subscription>`
- Resource Group: `/subscription/<subscription>/resourceGroups/<resource group>`
- Note: Wildcards (\*) are not allowed in the scope.

### Actions

- `Microsoft.Authorization/*`: Management of access permissions (role assignment, access review, policy) for a resource group.

## Role Cloning

Cannot be Cloned: 
- Built-in Azure AD roles (roles in EntraID)

Can be Cloned:
- Built-in subscription roles (roles in IAM)
- Custom roles whether in EntraID / IAM

## Endpoint policy

Service Endpoints use private IP addresses to access Azure services, simplifying access control with ACLs.

For example, 

- Normally, a VM accessing a storage account is seen from a public IP address.
- With a Service Endpoint and policy, the VM uses a private IP, allowing firewall rules to restrict access.

While Service Endpoints can be used with various Azure services, endpoint policies currently offer extra control only for Storage Accounts.

- VNET and Service Endpoint Policy must be in the same region.
- Without a policy, a subnet can access any storage account.
- Policies can cover:
  - All accounts in a subscription
  - All accounts in a resource group
  - A single account
- One service endpoint can have multiple policies.

## Azure Policy

Policies can be assigned to:

- Root Management Group
- Management Groups
- Subscriptions
- Resource Groups
- Resources

We can specify that a policy applies to some scopes except for specific ones. However, the Root Management group cannot be selected for the exclusion.

Policies are cumulative and follow the most restrictive rule.

- Each assignment is independently evaluated.
- The net result is considered to be cumulative most restrictive

When multiple policy assignments affect a resource:
- Each assignment is independently evaluated.
- The net result of policy is considered to be cumulative and the most restrictive outcome is applied.

Regarding deny and allow policies:
- If a policy has a deny effect, it will block the specified action, even if other policies allow it.
- There is no explicit `allow` effect in Azure Policy. By default, all actions that are not explicitly denied are allowed.

If the policy has a explict `allows` effect, 
- only those actions are permitted
- all other actions not explicitly allowed are implicitly denied

If the policy has a explict `denies` effect, 
- only those actions are blocked.
- all other actions not explicitly denied are implicitly allowed

Example:
- Root tenant level allows all actions except creating a virtual network.
- Subscription level allows creating a virtual network.
 
Result: You cannot create a virtual network because the root tenant's restriction takes precedence.

Tags can be assigned to management groups, subscriptions, resource groups, and resources, but they do not inherit from the parent level.

For tag inheritance, we can use the built-in Azure policy. This policy ensures that when a resource is created or updated, it automatically inherits the tag.

There are two types of policies for tag inheritance:

1. Modify: Can trigger a remediation manual
2. Append: Can trigger a remediation when the resource is updated or created.

These policies ensure that tags are automatically assigned to resources as needed.

## Resource lock

There are two types of locks:

- **Read-only**: Prevents modifications to a resource, such as moving it in or out, while allowing other operations.
  
- **Delete**: Prevents deletion of a resource.

Locks can be applied to:

- Subscriptions
- Resource groups
- Resources

However, locks cannot be applied to management groups.

## SSPR (Self-Service Password Reset)

Enable non-administrative users to reset their own passwords.

Administrators do not adhere to the SSPR policy.

Capabilities of the Global Administrator include:

- Enabling/disabling SSPR
- Resetting passwords for administrative users
- Setting up authentication methods
- Configuring security questions

Capabilities of the User Administrator include:

- Resetting passwords for non-administrative users
- Viewing SSPR reports

## Azure Device

Azure AD supports registration and management of various devices, including:

- Windows 10 or 11 laptops
- iPhones and iPads
- Android phones and tablets
- MacOS devices

There are two primary ways to add a device to Azure AD:

1. **Device Registration**: For personal or BYOD (Bring Your Own Device) use.
2. **Device Join**: For company-owned devices.

Azure devices enable:
- Single Sign-On (SSO) to cloud and on-premise resources.
- Implementation of Conditional Access policies. (e.g.: allowing specific app access based on registered device criteria)

## Microsoft Intune

Microsoft Intune is primarily used for:

- Mobile Device Management (MDM): Managing security and settings on mobile devices.
- Mobile Application Management (MAM): Controlling access to and security of mobile applications.
