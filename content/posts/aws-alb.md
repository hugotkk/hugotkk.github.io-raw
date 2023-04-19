---
title: "AWS ALB"
date: 2023-04-19
tags:
- cloud
- aws
- devops
- alb
---

## Two connections maintained
* ALB to target servers.
* Client to ALB.
## Keepalive/h2 setting
* Applies to ALB-to-target connection, not client-to-target servers.
* Why Keepalive help to improve the performance:
    * Limited number of connections between ALB to target servers, keepalive helps.
## Sticky session
* Cookie-based.
## Idle_timeout
* Applies to both ALB-to-target and client-to-ALB connections.
* Closes connections without data sent/received within timeout period.
## Timeout configuration
* ALB idle_timeout should be longer than the application timeout.
* Prevents traffic being sent to dead targets and causing 502 errors.
