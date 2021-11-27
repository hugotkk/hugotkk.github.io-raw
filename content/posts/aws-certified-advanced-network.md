---
title: "AWS Certified Advanced Networking - Specialty"
date: 2021-11-26
tags:
- cert
- exam
---

# VPC Addressing

## Reversed IPs in subnet

https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html


## Default route table will have rule: <vpc_cidr> local

## Limited ips per ENI

Single ENI can have multiple private ip. Depends on the instance types.

https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html

# VPC Route tables

by default the subnet will use vpc route table

we can customize the route table per subnet

# VPC Firewall 

EC2 -> Security Group -> NACL

Security Group is stateful

# DC

need to know about BGP first
