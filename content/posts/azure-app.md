---
title: Azure Enterprise Application
date: 2024-07-15
tags:
  - azure
  - enterprise application
---

## Enterprise Application

Enterprise Applications in Entra ID serve as a bridge between various resources and identities.

- On-premise app to Azure resource: Uses service principal
- On-premise resource to Azure resource: Uses service principal
- User to Azure resource: Uses user or group
- Azure resource to Azure resource: Uses managed identity

### Configure Enterprise Application

1. Create the app in Entra ID
2. Grant permissions to the service principal (the app)
3. Create a client ID and secret
4. Use these credentials in the app to access Azure resources

### SSO Options

#### SAML-based
For applications that support SAML, like AWS Console:
- Create an enterprise app in Azure AD
- Configure SAML metadata on both sides

#### Password-based
For applications that only support username/password authentication:
1. Create an enterprise app with password-based SSO
2. Azure AD will analyze the app's login page
3. Add users and input their app credentials
4. When users log in with Azure AD, it uses the stored credentials to authenticate to the app

### Provisioning Service

- Azure AD can automatically sync user accounts to the enterprise application
- Admins still need to assign permissions to users in their app
- Azure AD provisioning service can provision Azure AD accounts to enterprise applications (e.g., Salesforce)

### Authentication Flows

#### Authorization Code Grant
- User-interactive (e.g., social media login)
- Used when acting on behalf of the user

#### Client Credentials Grant
- Programmatic, application-to-application communication
- Used when acting on behalf of the application

For application-to-application scenarios (like accessing Key Vault), the Client Credentials Grant Flow is more appropriate.

### Permission Types

1. Delegated Permissions:
   - App gets access token with user's data
   - Example: App1 accessing User1's email

2. Application Permissions:
   - App gets access token with application's data
   - Example: App1 accessing App2's roles

Application permissions are typically used for background services or daemons where there's no user interaction.