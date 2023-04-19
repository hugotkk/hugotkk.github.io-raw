---
title: "Understanding OAuth 2.0: Explore with Amazon Cognito"
date: 2023-04-06
tags:
- cloud
- aws
- cognito
- oauth2
- oidc
- iam
---

## Difference between OAuth2.0, OIDC and SAML2

* OAuth 2.0 provides authorization using ID token.
* OIDC provides authentication using access token.
* SAML2 provides both authentication and authorization.
### OAuth 2.0 
  * allows users to log in, agree to the OAuth permission grant, and generate an access token (like an API key).
  * Access tokens have permission to access the API (resource server), but they are not related to the user's identity.
  * Access tokens can be renewed using a refresh token.
  * ID tokens can show who the user is.
  * For example, when logging into a website with Google, Google will generate an access token for the user, which the website can use to call APIs to get the user's email and name. The browser cookie will link the user with the access and refresh token.

## Amazon Cognito


- User pools can integrate with ALB and API Gateway.
- Identity pools can manage access control on AWS resources such as S3 and DynamoDB.
- User mapping can associate IAM roles with specific users or groups.
- By default, user mapping maps into authenticated and unauthenticated IAM roles.
- Rules can be configured to assign specific IAM roles based on conditions (e.g., specific attributes or group membership).
- With an ID token obtained through Amazon Cognito, temporary AWS access keys and secrets can be obtained using `aws get-credentials-for-identity`. These AWS credentials can be used to access AWS resources.

### Authentication Process
  - Clicking the login button and redirecting to the authorize URL.
  - Entering login credentials on the Cognito login page.
  - Redirecting to a specified redirect URL upon successful authentication.
  - Making additional requests to obtain access tokens and refresh tokens depending on the grant type.
  - Using the authorized code obtained from the initial request to obtain access tokens and refresh tokens if the grant type is `authorization_code`.
  - Including the `response_type=token` parameter in the authorization request if the grant type is implicit, which returns the access token and refresh token directly in the response without additional requests to the token URL.

### Authenticated Role Selection
  - Amazon Cognito user pools come with two default roles: Unauthenticated and Authenticated.
  - Role selection allows for different roles to be assigned to authenticated users.
  - "Choose role with rules" allows for rules to be set based on the JWT.
  - The following attributes can be referenced in the JWT: phone, email, username, default attributes, and custom attributes.
  - To reference the default and custom attributes in the IAM policy, the authorize scope needs to include "profile".
  - Users are allowed to be assigned to groups, and each group can be assigned multiple IAM roles.
  - `cognito:roles` and `cognito:preferred_role` will be added to the JWT token when "Choose role from token" is used to grant permission to user.
  - `cognito:groups` can also be found in the JWT if the user is in a group.

## Lab1: Cognito Identity Pool

### What I Need

- Cognito User and Identity Pool
- IAM roles for authenticated and unauthenticated user
- Both roles grant `PutObject` right to the S3 bucket
- S3 bucket

### Upload Object to S3 Bucket with Cognito

#### With AWS CLI

https://gist.github.com/hugotkk/bf3daf2148d9bc82303f62cb360e6401

- Get ID token from Cognito
- Get AWS credentials from Cognito API
- Use the credentials to upload, download, or presign an S3 object

#### With Cognito and AWS JavaScript SDK on Browser

- Similar to AWS CLI, but on a web page
- Need additional CORS policy to allow object update from the webpage
- The Python script will start a simple HTTP server and serve the file

https://gist.github.com/hugotkk/21385eb08a366b587f1f434eff43f381

## Lab2: Authenticated role selection

- Create a new user in the Cognito user pool
- In the IAM roles for authenticated and unauthenticated users, remove S3 permissions
- Create a new IAM role with S3 permissions
- For "Choose role with rules" testing:
  - Assign the attribute `profile: admin` to the first user
  - Create a new rule that checks if the claim has `profile` and is equal to `admin`, and assigns the new IAM role
- For "Choose role from token" testing:
  - Create a new group and add the first user to the group
  - Attach the new IAM role to the group
- Test by uploading an object using login.html (code provided in previous example)
- Expected result: The first user should be able to update objects in S3, while the second user should not be able to.
