---
title: Azure Application Proxy
date: 2024-07-15
tags:
  - azure
  - entra connect
  - entra domain service
  - application proxy
---

## Entra Connect

It is a service that integrates on-premises AD with Azure AD.

Entra Connect Health can be configured to send notifications if there are any AD sync issues.

Pass-through authentication allows Azure AD to send authentication requests to the on-premises AD to authenticate users for on-premises applications.

## Entra Domain Service

It offers managed domain services such as LDAP and Kerberos authentication in the cloud.

We can create an Entra Domain Service within a VNET

In case Domain Services are already running on-premises and want to be integrated with Azure:
- Synchronize the on-premises AD with Azure AD using Entra Connect
- Create the Entra Domain Service

## Application Proxy

This allows us to securely access the on-premises web applications.

### Setup
- Configure Entra Connect
  - Sync on-premises AD with Azure AD
  - Enable Pass-through Authentication to allow users to authenticate against on-premises AD
- Create Application Proxy
  - Setup Application Proxy
  - Install the Application Proxy Connector on a server in on-premises environment. This connector will handle the communication between Azure AD and on-premises applications
- Register Enterprise Application
  - Register the on-premises application as an Enterprise Application
- Conditional Access Policies
  - To limit user access based on specific conditions (e.g., location, device compliance) or to enforce MFA, configure Conditional Access Policies in Azure AD
  - Apply the policy to the Enterprise Application

### Login Flow

- User attempts to access the on-premises application via Application Proxy from public internet.
- The request is routed through the Application Proxy Connector
- The Connector will authenticates the user against the on-premises AD using Pass-through Authentication
- Once authenticated, the connector forwards the request to the on-premises application
- The user gains access to the application

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