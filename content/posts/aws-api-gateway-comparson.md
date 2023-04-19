---
title: "Comparing HTTP and REST API Gateways"
date: 2023-04-19
tags:
tags:
- cloud
- aws
- devops
- api-gateway
---

## Main difference

|                            | REST                                            | HTTP                          |
| -------------------------- | ----------------------------------------------- | ----------------------------- |
| Quota Management           | Per group                                       | Not supported                 |
| API key Management         | Supported                                       | Not supported                 |
| Authorization              | Lambda/Cognito                                  | Lambda/IAM/JWT                |
| Lambda Input               | Payload only                                    | Request details (Event)       |
| VTL model                  | Supported                                       | Not supported                 |
| SDK and Documentation Generation      | Supported                                       | Not supported    

## Lambda integration

Both REST and HTTP Lambda integrations offer a powerful and flexible way to integrate Lambda functions with API Gateway, with some differences in input/output format and response handling.


### Input / Output Format
- Lambda Proxy Integration in REST and Lambda Integration in HTTP both pass the request information to the event object in the Lambda function. However, the input and output format are different.
    - [REST Lambda Proxy Integration Input Format](https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html#api-gateway-simple-proxy-for-lambda-input-format)
    - [HTTP Lambda Integration Proxy Format](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-develop-integrations-lambda.html#http-api-develop-integrations-lambda.proxy-format)
- In HTTP APIs, if the Lambda function returns valid JSON and doesn't return a `statusCode` or a string, API Gateway assumes that the result is the body of the response. In REST APIs, the Lambda function has to construct the response object with the appropriate status code, headers, and body. This is not possible in HTTP APIs.

### VTL
- In REST APIs, the Lambda function can use VTL templates to transform the response data. VTL templates allow for specifying how the response data should be transformed from one format to another. 
- In HTTP APIs, only JSON data is supported and response transformation using VTL templates is not possible.


\* The VTL is used in REST APIs for the following:
- Request & Response Validation
- Generate an SDK
- Initialize a mapping template


### Data Transformations

| Integration Type | Path/ Query/ Header | Mapping Template |
| --- | --- | --- |
| lambda | N/A | Y |
| lambda proxy | N | N |
| http | Y | Y |
| http proxy | Y | Y |

### Data that Backend Receives

| Integration Type | Path | Query | Headers | Payload |
| --- | --- | --- | --- | --- |
| lambda | N | N | N | Y |
| lambda proxy | Y | Y | Y | Y |
| http | Y | Y | Y | Y |
| http proxy | Y | Y | Y | Y |