---
title: "Building a Secure API Gateway with Cognito"
date: 2023-04-19
tags:
tags:
- cloud
- aws
- devops
- api-gateway
- cognito
---

To enable authorization the api with Amazon Cognito User Pools:
- Create a user pool. Check out [Secure your API Gateway with Amazon Cognito User Pools](https://www.youtube.com/watch?v=oFSU6rhFETk) for a video tutorial.
- In the "Method Request" > "Auth" section of the API Gateway console, select the user pool.
- Access the API with an ID token:
    ```
    curl --location 'https://<my_api_gateway_domain>/<my_api>' \
    --header 'Authorization: Bearer <id_token>'
        ```

To generate the ID token from Cognito:
- Get an authorization token with the authorize API: `https://<my_cognito_domain>/oauth2/authorize?response_type=code&client_id=<client_id>&redirect_uri=<redirect_url>&state=STATE&scope=openid%20email`
- Use the authorization token to exchange it for an ID token from the token API:
    ```
    curl --location 'https://<my_cognito_domain>/oauth2/token' \
    --header 'Content-Type: application/x-www-form-urlencoded' \
    --data-urlencode 'grant_type=authorization_code' \
    --data-urlencode 'client_id=<client_id>' \
    --data-urlencode 'client_secret=<client_secret>' \
    --data-urlencode 'code=<auth_token>' \
    --data-urlencode 'redirect_uri=<redirect_uri>'
        ```

For more details about the authorize and token endpoints, check out the following links:
- [Authorize endpoint](https://docs.aws.amazon.com/cognito/latest/developerguide/authorization-endpoint.html)
- [Token endpoint](https://docs.aws.amazon.com/cognito/latest/developerguide/token-endpoint.html)

A more straightforward way to obtain the tokens is by using Postman, which supports OAuth 2.0 authentication. Check out [OAuth2 0 Authorization with Postman](https://www.youtube.com/watch?v=pxD9e2fk9fE) for a video tutorial.