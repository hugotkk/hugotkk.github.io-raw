---
title: "ALB and API Gateway"
date: 2023-03-29
tags:
- cloud
- aws
- devops
- alb
- api-gateway3
- websocket
- grpc
---

# ALB

* Two connections maintained:
    1. ALB to target servers.
    2. Client to ALB.
* Keepalive/h2 setting:
    * Applies to ALB-to-target connection, not client-to-target servers.
* Sticky session:
    * Cookie-based.
* Why Keepalive help to improve the performance:
    * Limited number of connections between ALB to target servers, keepalive helps.
* Idle_timeout:
    * Applies to both ALB-to-target and client-to-ALB connections.
    * Closes connections without data sent/received within timeout period.
* Timeout configuration:
    * ALB idle_timeout should be longer than the application timeout.
    * Prevents traffic being sent to dead targets and causing 502 errors.

# gRPC

* Concept:
    * Similar to REST API and JSON.
* Payload format:
    * Binary
* Performance:
    * Faster as smaller size than JSON.
* Usage:
    * Primarily for service-to-service communication.
    * Browser-to-service not widely supported.
* Protocol:
    * Built on HTTP/2 and HTTPS only.

# WebSocket

* WebSocket characteristics:
    * Bi-directional, persistent TCP connection between client and server.
* Scaling limitations:
    * WebSocket is stateful, cannot horizontally scale without backend to store state.
* WebSocket API Gateway benefits:
    * Aids scaling by maintaining WebSocket connections.
    * Client and Lambda interact with API Gateway.
    * Lambda uses APIGatewayConnection API to send data to open connections.
    * Client send data to gateway directly.
* API Gateway limitations:
    * WebSocket connection lasts for 2 hours.
    * Idle timeout is 10 minutes.

