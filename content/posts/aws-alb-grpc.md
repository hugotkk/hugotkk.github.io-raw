---
title: "Setup gPRC with AWS ALB"
date: 2023-04-19
tags:
- cloud
- aws
- alb
- grpc
---

### gRPC

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
    * Built on HTTP/2
    * client -> ALB: HTTPS only; ALB -> target: HTTP/HTTPS
