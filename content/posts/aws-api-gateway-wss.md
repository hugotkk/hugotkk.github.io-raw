---
title: "Building a Chat Room with Lambda and Websocket in AWS API Gateway"
date: 2023-04-19
tags:
tags:
- cloud
- aws
- devops
- api-gateway
- websocket
- dynamodb
- lambda
---

## WebSocket Overview

* WebSocket Characteristics
  * Bi-directional, persistent TCP connection between client and server
* Scaling Limitations
  * WebSocket is stateful, cannot horizontally scale without a backend to store state (eg: Redis)

## API Gateway with Websocket

* Benefits
  * Aids scaling by maintaining WebSocket connections
  * Serverless - Client and Lambda interact with API Gateway
  * Lambda uses APIGatewayConnection API to send data to open connections
  * Client sends data to gateway directly

* Limitations
  * WebSocket connection lasts for 2 hours
  * Idle timeout is 10 minutes

## Lab: Building a Chat Room

### Setup

I watched this tutorial on [how to build a chat using Lambda, WebSocket, and API Gateway with Node.js](https://www.youtube.com/watch?v=BcWD-M2PJ-8)

Access the code in this [GitHub Gist](https://gist.github.com/hugotkk/ce89d68704cf66a6338087f4352fb450)

* Server Setup
  * server.py is a simple HTTP server with Python to serve the client-side code

* Lambda Function
  * index.mjs is the Lambda code for the chat room
  * Implements `$connect`, `$disconnect`, `join`, `leave`, and `sendAll` functions
  * Uses DynamoDB to store `connectionIds` and `roomId` with the `connectionIds` that joined

### Common Errors and Solutions

#### Global Variable Persistence
- The video stores the connections and rooms data in global variables in lambda.
- Initially, it was thought that it would not work because each invocation is stateless.
- However, variables in the global scope can be shared between Lambda invocations.
- This is mentioned in the [Comparing the effect of global scope](https://docs.aws.amazon.com/lambda/latest/operatorguide/global-scope.html).
- However, it is more reliable to use a database for storing data.
- The use of global variables should be limited to demo purposes only and should never be used in the real world.

#### Connection Endpoint

- The endpoints can be found in the AWS Console under "API Gateway" > "Stages".
- One is WebSocket, starting with wss://, and another is HTTPS, starting with https://.
- When posting the data back to the client from Lambda, use the HTTPS endpoint but remove @connections at the end.
- For example, if the endpoint on the console is `https://d6sq1c52q5.execute-api.us-east-1.amazonaws.com/production/@connections`, we should omit the `@connection` when using `PostToConnectionCommand`.

#### Lambda Permissions

* Lambda needs permission to call invoke and manage connection
* Use `AmazonAPIGatewayInvokeFullAccess` or create a custom policy

Custom policy example:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "execute-api:Invoke",
                "execute-api:ManageConnections"
            ],
            "Resource": "arn:aws:execute-api:us-east-1:059035646743:d8ck9i69wa/production/*""
        }
    ]
}
```

Refer to the [AWS documentation](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-websocket-control-access-iam.html) for more information.


#### "Going Away" Exception

* You might encounter a "Going away" exception if you try to post data to a connection in `$connect`
* This is not supported, as mentioned in this [Stack Overflow post](https://stackoverflow.com/questions/67834850/goneexception-when-calling-post-to-connection-on-aws-lambda-and-api-gateway)


#### Using Node.js 18 in Lambda Function

* If you use Node.js 18, `require('aws-sdk')` will not work due to the use of ES module style
* When connecting to the WebSocket, it may return a 502 error, which is also logged in the API Gateway access log but not useful for debugging purposes.
* To diagnose the issue, check the Lambda error log on CloudWatch

To tail the CloudWatch log, use the following command:

```
aws logs tail <log_group> --follow
```

Refer to the [AWS CLI documentation](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/logs/tail.html) for more information.

