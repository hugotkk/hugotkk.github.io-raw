---
title: "AWS Certified Advanced Networking - Specialty"
date: 2021-11-26
tags:
- cert
- exam
---

# Resources

* [AWS Certified Advanced Networking Specialty 2021](https://www.udemy.com/course/aws-certified-advanced-networking-specialty-ans/)

# How to connect VPCs

* [Transit gateway peering attachments](https://docs.aws.amazon.com/vpc/latest/tgw/tgw-peering.html)
* [What is VPC peering?](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html)
* [Transit Gateway vs VPC peering](https://docs.aws.amazon.com/whitepapers/latest/building-scalable-secure-multi-vpc-network-infrastructure/transit-gateway-vs-vpc-peering.html)

| Solution / Situation | same region | cross region | cross account |
| ---                  | ---         | ---          | ---           |
| VPC Peering          | OK          | OK           | OK            |
| TGW Peering          | OK          | OK           | OK            |
| TGW (attach VPC)     | OK          | NO           | OK            |
| AWS Managed VPN      | OK          | OK           | OK            |
| EC2 Based VPN        | OK          | OK           | OK            |

Private Link is something like tgw peer but for spefic service only. It can cross region and account.

#  Scope

| Resource | Scope  |
| ---      | ---    |
| VGW      | VPC    |
| TGW      | VPC    |
| NLB      | VPC    |
| ALB      | VPC    |
| Route53  | VPC    |
| NAT      | Subnet |

# Exam Essentials

|    Page | Topic          |
|     --- | ---            |
|     190 | VPC            |
| 300-302 | VPC Peering    |
|     402 | VPN            |
|     632 | Direct Connect |

# VPC Peering

## Failed cases

| Page | Title                              |
|  --- | ---                                |
|  207 | CIDR overlap or transitive routing |
|  208 | from DX or VPN                     |
|  209 | to IGW                             |

# Site-to-site VPN with VGW

## Use cases

| Page | Title                                      |
|  --- | ---                                        |
|  335 | Site-to-site Connection with NAT-Traversal |
|  361 | Single Site-to-site Connection with VGW    |
|  363 | Multiple Site-to-site Connection with VGW  |
|  365 | Redundant VPN connections for HA           |

### VPN CloudHub

* 369

### EC2 Based VPN

| Page | Title                                   |
|  --- | ---                                     |
|  375 | Single instance                         |
|  376 | HA                                      |
|  378 | Horizontal scaling - VPN EC2 per subnet |
|  379 | Horizontal scaling - Split traffic      |

### Transit VPC

| Page | Title                                    |
|  --- | ---                                      |
|  391 | Transit VPC                              |
|  398 | global regions with single Transit Hub   |
|  398 | global regions with multiple Transit Hub |

## Failed cases

| Page | Title                                    |
|  --- | ---                                      |
|  340 | Site-to-site VPN to IGW                  |
|  341 | Site-to-site VPN to NAT                  |
|  343 | Site-to-site VPN to VPC Peering          |
|  344 | Site-to-site VPN to VPC Gateway endpoint |

## Successful cases

| Page | Title                                          |
|  --- | ---                                            |
|  342 | Site-to-site VPN to NAT Instance               |
|  345 | Site-to-site VPN to VPC Interface endpoint     |
|  345 | Site-to-site VPN to on-premise NAT to Internet |


# Direct Connect

## Use cases

| Page | Title                                    |
|  --- | ---                                      |
|  591 | VPN as a backup                          |
|  592 | Dual Devices                             |
|  593 | Dual locations                           |
|  594 | Dual locations with DX connection backup |
|  602 | VPN over DX                              |

## Failed cases

| Page | Title                                   |
|  --- | ---                                     |
|  530 | DX Gateway with multiple customer sites |
|  542 | DX Gateway with multiple TGWs           |

## Successful cases

| Page | Title                                   |
| ---  | ---                                     |
| 546  | DX Gateway with multiple customer sites |

# Limitation

[VPC Limit](https://docs.aws.amazon.com/vpc/latest/userguide/amazon-vpc-limits.html)

## Peering

VPC Peering: 125

## Routing

| Resource     | Limit |
| ---          |   --- |
| Private VIFs |   100 |
| Public VIFs  |  1000 |

## MTU (159)

| Resource     | Limit |
| ---          |   --- |
| VPC          |  9001 |
| VPC Peering  |  1500 |
| DX           |  9001 |
| TGW (to DX)  |  8500 |
| TGW (to VPN) |  1500 |
| VPC Endpoint |  1500 |
| NAT          |  1500 |
| IGW          |  1500 |
| VPN          |  1500 |

## Bandwidth

* [Quotas for your TGW](https://docs.aws.amazon.com/vpc/latest/tgw/transit-gateway-quotas.html)
* [Aggregated throughput limit for VGW](https://aws.amazon.com/vpn/faqs/)
* [NAT gateways](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) 

### VPC (180)

| Resource    | Limit         |
| ---         | ---           |
| VPC Peer    | no            |
| IGW         | no            |
| NAT         | 5-45Gbps      |
| TGW         | depends       |
| VGW         | 1.25Gbps      |
| VGW (to DX) | depends on DX |

### TGW

* 1 VPN can have 2 tunnels

| Resource   | Limit per resource |
| ---        | ---                |
| VPC        | 50Gbps             |
| DX         | 50Gbps             |
| TGW        | 50Gbps             |
| VPN tunnel | 1.25Gbps           |

### VPN Tunnel

1.25Gbps

### EC2

* [100G networking in AWS, a network performance deep dive](https://toonk.io/aws-network-performance-deep-dive/index.html)

EC2 network performance rules:
* single flow limit
* within regions and other

| Situation       | Limit |
| ---             |   --- |
| w/i the region  |  100% |
| to other regions |   50% |
| igw             |   50% |
| dx              |   50% |

#### Single Flow

| Situation           | Limit  |
| ---                 | ---    |
| w/i placement group | 10Gbps |
| other               | 5Gbps  |

#### Aggregated

* [General Purpose Network Performance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/general-purpose-instances.html#general-purpose-network-performance)

| Situation               | Limit   |
| ---                     | ---     |
| w enhanced networking   | 100Gbps |
| w/o enhanced networking | 25Gbps  |

| Network Driver | Limit   |
| ---            | ---     |
| Intel 82599VF  | 10Gbps  |
| ENA            | 100Gbps |
| EFA            | 100Gbps |


# VPC Addressing

## Reversed IPs in subnet

https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html

## Default route table will have rule: <vpc_cidr> local

## Limited ips per ENI

Single ENI can have multiple private IPs. Depends on the instance types.

https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html

# VPC Route tables

by default the subnet will use vpc route table

we can customise the route table per subnet

# VPC Firewall 

EC2 -> Security Group -> NACL

Security Group is stateful

# DC

need to know about BGP first

# ECMP 

* can aggregate the VPN tunnel
* only support at BGP routing

