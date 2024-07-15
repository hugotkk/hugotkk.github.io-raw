---
title: Azure AD
date: 2024-07-11
tags:
  - azure
  - ad
  - license
  - device
---

## Users

If a user is synced from an external source, JobTitle cannot be modified. But UsageLocation can be updated.

### Bulk Operations

Create a CSV file contains the necessary fields

Bulk Create Users
- displayName
- userPrincipalName
- passwordProfile
- accountEnabled

Bulk Delete Users
- userPrincipalName

## External Users

Guest Accounts (B2B - Business to Business):
- Allows external users to access your organization's resources
- Users can use their existing credentials from their home organization

Cloud-only Users (B2C - Business to Consumer):
- Requires creation of a new account within your tenant
- Cannot use existing credentials from another organization

Both B2B and B2C configurations maintain a single-tenant application
External users are represented within your tenant using a format like hugo#org2@org1
This approach allows referencing external users within our own tenant structure

### Guest User Invitation

"Unable to invite user" error often indicates lack of permissions

To resolve,

- Go to EntraID > Users (or External Identities)
- Navigate to External Collaboration > Guest invite settings
- Change settings to allow Admin1 to invite guest users

### Bulk Operations

Create a CSV file contains the necessary fields

Bulk Invite:
- inviteeEmail
- inviteRedirectURL

## Groups

### Group Types

Microsoft 365 Groups
- Collaboration-forced
- Shared Mailbox, Calendar, Files, SharePoint site
- Memebership can be Users only
- Can apply Expiration Policy

Security Group
- Access management for app, resources and licenses
- Memebership can be Users / Devices / Service Principal / Groups
- RBAC in Azure, License Management

### Memberships

Dynamic Membership
- Uses query-based rules to automatically add/remove members
- Supports both user and device attributes (security groups)

Assigned Membership
- Members are added or removed manually

Both Microsoft 365 and Security groups support dynamic and assigned memberships.

## Devices

### Adding Admin User

Adding an Admin User to All Domain-Joined Computers:
1. Log in to the Azure portal with a user account as Device Admin or Global Admin
2. Go to Entra ID > Devices > Device Settings > Manage additional local administrators on all Azure AD joined devices
3. Click on "Select" to choose the users or groups we want to add as local admins

### Device Join Types

Registered
- Persional Device
- Limited access to organizational resources

Joined
- Company-owned Device
- Full access to organizational resources

### Group Management

- Group can contain both registered / join devices
- Dynamic membership rules can be used to automatically add devices to groups based on attributes

### Azure RBAC

User Admin:
- Can manage user membereships
- Cannot manage device memberships

Cloud Device / Intune Admin:
- Can enable / disable / delete / read device keys
- Cannot add device to groups

Owner:
- full control

### Local Admin (Windows)

Local administrators on an Azure AD-joined device include:
- The user who joined the device (Owners)
- Global Admin
- Users / Group that specified at "Manage Additional local administrators on all Azure AD joined devices"

### Common Settings

Users may join devices to Azure AD

- Controls who can join devices to Azure AD
- Go to Device settings > All, Selected, None
- Default is "All"

Modify the max number of devices per user

- Limits how many devices a single user can join or register

## Licenses

- Licenses assigned to a group only apply to direct members of that group.
- Members of nested groups do not inherit the license.

### Available Group Types

Licenses can be assigned to:
- Security groups
- Microsoft 365 groups with security enabled

### Deletion

Group
- Before deleting a group, all assigned licenses must be removed from the group.

User
- Users can be deleted regardless of their license status.
- No matter the license is directly assigned or inherited from a group.

### Microsoft Defender

- License can be assigned directly to users

## Cost Management

1. Assign tags to resources
2. Filter the cost analysis view by tags.
3. Download the usage report

## Alert Notifications

Separate email will be sent when the notification was set at
- Action Group
- Alert recipients