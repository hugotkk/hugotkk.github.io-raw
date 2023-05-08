---
title: "Unexpected Limitations: AWS Lambda's Inability to Directly Reference Secrets Manager and Parameter Store"
date: 2023-04-19
tags:
- cloud
- aws
- lambda
- secret-manager
- ecs
---

I was surprised to find that AWS Lambda cannot directly reference records from Secrets Manager in environment, especially considering that ECS can reference records from both Parameter Store and Secrets Manager. There are two ways to overcome this limitation in Lambda:

1. Use the AWS API.
2. Use a [Lambda Extension](https://docs.aws.amazon.com/secretsmanager/latest/userguide/retrieving-secrets_lambda.html) to retrieve secrets from Secrets Manager.
