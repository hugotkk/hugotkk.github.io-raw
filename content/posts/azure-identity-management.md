---
title: Azure Identity Management
date: 2024-07-14
tags:
  - azure
  - ad
  - conditional access
  - entitlement management
  - access review
  - privilege identity management
---

## Identity Protection

### Conditional Access

Purpose to enforce specific controls on access to apps based on certain conditions.

For example, MFA enforcement can be found under the Grant control section.

- Assignments > Users / Groups
- Target Resource > Azure Management, Office 365, Cloud APP
- Conditions > Risks, Locations OR Access Control > Grant > Require MFA

## Identity Governance

### Entitlement Management

Entitlement Management works with Azure AD B2B to govern access for users from other organizations.

- Administrators create access packages containing resources like applications, SharePoint sites, or Azure AD groups.
- Define policies for who can request access, approval requirements, and access review settings
- Manage access lifecycle with automatic expiration and extensions. For example,
  - Remove the access assignment after 365 days
  - Remove the users after 30 days if they have no assigned access

- Guest users can request access to the defined access packages through the Access portal.
- The request goes through the approval process defined in the access package policy.
- Upon approval, the guest user is granted access to the resources in the access package.

### Access Review

- Available at Azure AD Premium P2 or Microsoft 365 E5 licenses
- Dynamic group does not support in access review
- Nested groups / Synced user
  - can be reviewed
  - Actions won't take effect on these members

Scopes:
- Guest users only
- Specific Groups
- Non-Administrative users

### Privilege Identity Management (PIM)

- Access reviews for privileged roles (Like User Admin)
- Just-in-Time (JIT) privileged access (temp firewall rules)
- Alerts for suspicious or unauthorized activities
