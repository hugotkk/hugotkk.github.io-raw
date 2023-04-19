---
title: "Understanding TLS Passthrough and Other Features of AWS NLB"
date: 2023-04-19
tags:
tags:
- cloud
- aws
- devops
- alb
- tls
- nlb
---

## TLS Passthrough with AWS NLB

To setup TLS passthrough with NLB, follow these steps:

* Listen: TCP - can be 80 or 443
* Target: TCP - 443
* Backend: HTTPS on 443

Here are some interesting features of NLB:

1. Proxy Protocol v2
    - TCP 
    - Adds a binary header before the HTTP payload 
    - Provides the information about the client like X-Forward-For in ALB
    - To enable Proxy Protocol v2, the target server has to understand the protocol
    - Health check has to be TCP as well
    - Nginx can listen to Proxy Protocol v2 - [config](https://gist.github.com/hugotkk/a2e879cbda87df5138790c21bfa44739) available
    - The backend is a simple HTTP server -> it can see the X-Forward-To headers that were passed in the request
1. No security group policy on NLB
1. Health check
    - HTTPS: 443 with invalid cert -> OK
    - HTTP: if HTTP redirect to HTTPS was set up, the health check status will be 301 
    - Can check the health check request in the server access log
    - If 400 - wrong protocol. H2 or sending HTTPS to HTTP port
    - If no request comes in - security group blocked - in ALB or EC2
    - Can shorten the health threshold and interval to speed up the health check initially
1. Preserve client IP addresses
    - Can preserve client IP. In PHP, the `$_SERVER['REMOTE_ADDR']` will change to the client's IP
1. Zone level load balancing
    - By default, NLB load balances within the zone
    - Imagine that the NLB is a group of EC2 in different zones. The CNAME has to return multiple IPs to each EC2
    - The route algorithm is affected on the load balancer EC2 zone itself. Not cross-zone, but ALB is cross-zone by default

