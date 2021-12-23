---
title: "AWS Certified Advanced Networking - Specialty"
date: 2021-11-26
tags:
- cert
- exam
- aws
- networking
---

# Resources

* [Exam Landing Page](https://aws.amazon.com/certification/certified-advanced-networking-specialty)
* [Sample Questions](https://d1.awsstatic.com/training-and-certification/docs-advnetworking-spec/AWS-Certified-Advanced-Networking-Specialty_Sample-Questions.pdf)
* [Exam Guide](https://d1.awsstatic.com/training-and-certification/docs-advnetworking-spec/AWS-Certified-Advanced-Networking-Specialty_Exam-Guide.pdf)
* [udemy course](https://www.udemy.com/course/aws-certified-advanced-networking-specialty-ans/)
* [udemy mock exam](https://www.udemy.com/course/aws-certified-advanced-networking-specialty-ans/)
* [whizlabs test & hand-on labs](https://www.whizlabs.com/learn/course/aws-advanced-networking-speciality)
* [Network to Amazon VPC connectivity option](https://docs.aws.amazon.com/whitepapers/latest/aws-vpc-connectivity-options/network-to-amazon-vpc-connectivity-options.html)

# TODO

* take notes
* revision
* mocks (by topic)
* whizlabs
* revise udemy quiz + exercise
* mock 1 (udemy)
* mock 2 (whizlabs)

# Revision 

The numbers below are the page no of the udemy course's pdf: AWS Certified Networking Specialty Slides v1.1.

## Summary

* DNS (151)
* Advanced Networking (188)
* VPC Endpoint (292, 298)
* Site-to-site VPN (347, 401, 407)
* VPN Tunnels and Routing (348, 352)
* DX Gateway with VGW (532)
* DX with TGW (547)
* DX Billing (624)
* DX (630)
* Troubleshooting in DX (626)
* Gateway Load Balancer (816)

## Q & A
ALB (677)
DX (625)

## Remember

## Good to know

* SG (46)
* BYOIP (90)
* VPC Traffic Mirroring (105)
* DHCP Option Sets (124)
* TGW (251)
* AWS Site-to-Site VPN (403)
* Network Load Balancer (656)

## Comparisons

* private, public and EIP (33)
* IPv4, IPv6 (36)
* NACL, SG (48)
* NAT Gateway, Instance (64)
* VPC Peering vs Transit Gateway
* Cloudfront function vs lambda@edge (703)
* AWS global accelerator vs Cloudfront (711)

## Exam Essential

* VPC Fundamentals (67)
* Advanced VPC (190)
* VPC Peering Endpoint (301)
* AWs Site-to-Site VPN (402)
* Direct Connect (632)
* Firewall (806)
* Gateway Load Balancer (817)

## Revision by topic

### VPC Fundamentals (17)

* What is OSI? (306)
* How to calculate the no of address in a CIDR (like 192.168.0.1/28)? (18)
* Can we override the main route table in VPC? (26)
* Which IPs are reserved for a vpc? (29)
* What is the default behaviour of SG (inbound and outbound)? (46)
* How is the NAT gateway charged? (59)
* Can we apply SG on a NAT gateway? (59)
* What is the port range NAT gateway needs for outbound connection? (59)
* Why do we need NAT with HA(AZ)? (62)
* What is the most important setting to setup NAT with EC2? (63)

### Advanced VPC (72)

* What is the limitation of adding a secondary CIDR? (79)
* How many private, public and elastic ip and sg can one ENI have? (81, 85)
* What is the use case of dual home setup with multiple ENIs? (84)
* What are the prerequisites of BYOIP? (89, 90)
* What type of Flow Logs can we capture from a VPC? (92)
* Where does the Flow Logs Action field come from? (97)
* What type of record cannot be captured in a flow log? (98)
* What can be the source & target of a VPC Traffic Mirror? (102, 105)
* What port does the VPC Traffic Mirror require? (105)
* How to set a custom domain / dns on an EC2 instance? (113)

### VPC DNS and DHCP

* What does enableDNSSupport do? (117)
* What does enableDNSHostname do? (117)
* How to resolve the hostname in VPC peering and TGW? (132)
* How to resolve the hostname in Hybrid cloud (AWS to on-premise, on-premise to AWS) (old & new way)? (132-150)

### VPC Network Performance and Optimization

* Formula of throughput (154)
* What is Jumbo frame? (154)
* What is Path MTU Discovery? (155)
* What are the advantages of using placement group - cluster (162)
* What is the bandwidth limitation between VPC, EC2 instances, VPN and DX (179-183)?

### VPC Peering

* What is the limitation of VPC Peering? (207)

### TGW

* what attachments does TGW support? (211)
* What is the special route in TGW? (230) 

### VPC Endpoint

* How many types of VPC endpoint? What is the difference between them? (258)
* Can on-premises network access to the gateway endpoint with VPN/DX? (273)
* What kind of source can be used in a private link? (281)
* How to use hostname in interface endpoint? (288)
* How to use the interface endpoint without private DNS? (290)
* When should we use a private link? When should we use VPC Peering (297)

### AWS Site-to-Site VPN

* What port does IPsec need? (309)
* Range of public and private ASN (313)
* 4 common BGP Param for routing (319)
* What type of VPN does AWS support Site-to-Site VPN? (327)
* What routing method does AWS support in Site-to-Site VPN? (329)
* What port is required in VPN for NAT-T? (335)

### VPN Tunnels and Routing 

* How to set Active/Active tunnel (351)
* What is DPD? Port to send messages? default timeout? timeout action? (354)
* How to prevent a tunnel from terminating due to inactivity (355)
* How to monitor the tunnel status? (357)

### DX

* What is the fibre mode of DX? (450)
* what is the data link requirement of DX (450)
* What are the requirements for a customer router? (450)
* What is the port of BGP? (457)
* What is BFD? How to enable it? (457)
* How long does it take for failover with and without BFD? (458, 597, 598)
* How can a user whitelist the ips range from AWS? (459)
* How many VIF can a hosted connection have? (465)
* What Ip type does DX support? (498)
* What is the route prefix that public vif can advertise? (503)
* What is the route prefix that private vif can advertise? (507)
* What is the limitation of vif and vgw location? (507)
* What service cannot access inside a VPC with DX? (510)
* What is the usage of DX gateway? (517)
* How many vif can a DX connection have? (526)
* How many transit vif can a DX connection have? (547)
* How many vif can a DX gateway have? (530)
* How many vgw can a DX gateway have? (526,547)
* How many tgw can a Dx gateway have? (538)
* What is the charge for using Dx gateway? (533)
* How many routes can be advised per TGW through the DX gateway? (539)
* what is ECMP (251, 556) - only support BGP
* how to setup Active-Active with public vif and public ASN (558)
* how to setup Active-Active with public vif and public ASN (558)
* how to setup Active-Passive with public vif and public ASN (561)
* how to setup Active-Passive with public vif and public ASN (561)
* how to constrain the adviser routes to on-premises with public vif (567)
* how to constrain the adviser routes to aws with public vif (567)
* what is the routing policy with private vif (577, 582)
* What is LAG? (583)
* How many DX connections can a LAG have? (584)
* What is the bandwidth requirement of a LAG? (583)
* What is the operation connection attribute in LAG? (589)
* how to increase the resiliency of DX? (3 methods) (590)
* how to encrypt DX traffic? (2 methods) (601)
* When will the Dx connection be charged? (Hosted and Dedicated connection)  (621)
* What will be charged in Dx? (615)
* Who will be charged for the DTO fee? (622)

### ELB

* What Layer ALB, CLB, NLB, GLB  belong to? (639) 
* What protocol does ALB, CLB, NLB, GLB support? (641)
* Why does NLB have less latency than ALB? (652)
* What kind of target can be used in the ALB and NLB target group? (653)
* What kind of protocol is supported in the health check? (653)
* What are the connection idle timeouts of ALB, NLB, CLB? (657)
* What routing algorithm supported by ALB, NLB, CLB (659)
* How to keep the client ip in ALB, NLB (671, 672)
* How to keep the client on the same instance for a period of time? (661)
* What is Cross-Zone Load Balancing? (663)
* What is SNI and which ELB does it support? (667)
* What is Connection Draining (670)

### Cloudfront

* What is Cloudfront's origin? (683)
* Why public access is needed in cloudfront's origin (683)
* What is the origin group (687)
* How to change custom header / behaviour in Cloudfront? (698)
* How to restrict the content to specific geolocation? (696)
* What is AWs global accelerator? What does it work (709)
* What is the difference between AWS global accelerator and cloudfront? (711)

### Route53

* What is the longest and shortest TTL in route53 (721)
* What are the alias targets in route53 (724)
* How to bind an RDS DB instance in route53 (748)
* Routing Policies in Route53? (724)
* Which route53 performs the health check in a private VPC? (732)
* How to setup hybrid DNS (754)
* How does AWS ensure the HA in route53? (807)

### Network firewall

* What layer SG, NACL, network firewall, WAF, shield?
* What is the VPC level in SG and NACL? (766)
* When should we use nacl instead of sg? (768, 769)
* When should we use WAF instead of lacl ? (770)
* Stateless and stateful of sg, nacl, network firewall, shield? (773)
* How can a SYN cookie prevent DDos in packet flooding?

### Gateway Load balancer

* what is the use case of gateway load balancer (813)
* what port does gateway load balancer need for GENEVE (817)

### Other

Centralised VPC Interface endpoint 295

access s3 with endpoint
use a private link to access a web server (NLB/EC2)

Demo (266)

# Hand-on Labs

https://www.whizlabs.com/learn/course/aws-advanced-networking-speciality/195

## EC2
* [ ] 30

## CloudFront
* [ ] 1
* [ ] 2
* [ ] 10

## ALB
* [ ] 3
* [ ] 4
* [ ] 5

## WAF
* [ ] 11
* [ ] 12

## ACL
* [ ] 18
* [ ] 29

## VPN
* [ ] 21

## Endpoint
* [ ] 23
* [ ] 24
* [ ] 25

## Flow Log
* [ ] 26

## Container
* [ ] 28

# Udemy Sample Questions

https://www.udemy.com/course/practice-exam-aws-certified-advanced-networking-specialty/learn/quiz/5355502/result/633468092#questions

## Q2

priority of route table

https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html#route-table-priority-propagated-routes
https://docs.aws.amazon.com/vpn/latest/s2svpn/VPNRoutingTypes.html#vpn-route-priority

propagated < static
propagated < local

igw
nat
network interface
instance ID
tgw
vpc peering
gateway vpc endpoint
gateway load balancer endpoint


## Q3

route53 health check - the health check request is over internet

https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/health-checks-types.html

## Q7

need mesh peer network and private vif for each vpc

## Q9

misunderstand the question, asking about the DNS ips in the VPC

## Q10

don't understand the question...it is asking for abnormal behaviour but the answer is compliance only

at 325
* more specific prefix
* Highest Weight
* Highest Local Ref
* Shortest AS_PATH
* Lowest MED

NAT-T UDP4500 (335)

VPN Tunnel
* Active / Passive
* Active / Active

# whizlabs questions

## By Topics

### AWS Direct Connect

https://www.whizlabs.com/learn/course/aws-advanced-networking-speciality/195/quiz/15884/report/5741896

#### Q2

Some data need to be encrypted but some no need -> DX+VPN, DX 

#### Q4

QOS + VPN Instance (for something not supported by AWS, then use EC2 Instance)

#### Q7

BFD -> enabled by AWS by default (quick failure detection & fail over)

#### Q8

IPv4 is owned by the client. IPv6 auto assigned by AWS

### VPN

#### Q2

B <-WAN-> M <-VPN-> AWS

#### Q3

OK but less cost effective

ec2 vpn + dx ok
2 vpn + dx Ok
vpc peer OK
ec2 vpn OK

the keyword only needs connectivity on one application, so ec2 vpn is more cost effective

#### Q4

C D does not correct as VGW are not transitive

Remain A B. Difference between A B: use 2/1 ec2 vpn

## VPC

* Q1 -> misread VPN connection
* Q2 -> use reference instead of cidr
* Q5 -> VPC Peering does not need route53
* Q7 -> cannot use bucket policy on S3 endpoint as sourceIp are same
* Q8 -> no need to publish s3 then cloudwatch log. Flow Log: VPC or Network Interface

## SG

* Q1 -> SG cannot deny traffic. I forgot...
* Q5 -> default SG: allow b/w instances. allow outbound from instance

https://aws.amazon.com/certification/faqs/

# VPCs Connectivities

* [Transit gateway peering attachments](https://docs.aws.amazon.com/vpc/latest/tgw/tgw-peering.html)
* [What is VPC peering?](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html)
* [Transit Gateway vs VPC peering](https://docs.aws.amazon.com/whitepapers/latest/building-scalable-secure-multi-vpc-network-infrastructure/transit-gateway-vs-vpc-peering.html)

| Solution / Situation | same region | cross account | cross region |
| ---                  | ---         | ---           | ---          |
| VPC Peering          | OK          | OK            | OK           |
| TGW Peering          | OK          | OK            | OK           |
| TGW (attach VPC)     | OK          | OK            | NO           |
| AWS Managed VPN      | OK          | OK            | OK           |
| EC2 Based VPN        | OK          | OK            | OK           |

Private Link is something like tgw peer but for specific service only. It can cross regions and accounts.

#  Scope

| Resource | Scope  |
| ---      | ---    |
| VGW      | VPC    |
| TGW      | VPC    |
| NLB      | VPC    |
| ALB      | VPC    |
| Route53  | VPC    |
| NAT      | Subnet |

582 Routing

# Common Pattern

## Site-to-Site Connection routing

* Static - Active/ Active Tunnels (349)
* Static - Active/ Passive Tunnels (350)
* Dynamic - Active/ Active Tunnels (351)

## TGW

241: Centralised NAT gateway
243: Centralised NAT instance
244: Centralised NAT instance + VPN
245: Centralised VPC interface endpoint + VPN
247: Hybrid VPN
248: Hybrid DX

## VPC Peering

### Failed cases

| Page | Title                              |
|  --- | ---                                |
|  207 | CIDR overlap or transitive routing |
|  208 | from DX or VPN                     |
|  209 | to IGW                             |

## Site-to-site VPN with VGW

### Use cases

| Page | Title                                      |
|  --- | ---                                        |
|  335 | Site-to-site Connection with NAT-Traversal |
|  361 | Single Site-to-site Connection with VGW    |
|  363 | Multiple Site-to-site Connection with VGW  |
|  365 | Redundant VPN connections for HA           |

## Site-to-site VPN with TGW

| Page | Title                                     |
| ---  | ---                                       |
| 362  | Multiple Site-to-site Connection with TGW |

### Failed cases

| Page | Title                                    |
|  --- | ---                                      |
|  340 | Site-to-site VPN to IGW                  |
|  341 | Site-to-site VPN to NAT                  |
|  343 | Site-to-site VPN to VPC Peering          |
|  344 | Site-to-site VPN to VPC Gateway endpoint |

### Successful cases

| Page | Title                                          |
|  --- | ---                                            |
|  342 | Site-to-site VPN to NAT Instance               |
|  345 | Site-to-site VPN to VPC Interface endpoint     |
|  345 | Site-to-site VPN to on-premise NAT to Internet |

## VPN CloudHub

* 369

## EC2 Based VPN

| Page | Title                                   |
|  --- | ---                                     |
|  375 | Single instance                         |
|  376 | HA                                      |
|  378 | Horizontal scaling - VPN EC2 per subnet |
|  379 | Horizontal scaling - Split traffic      |

## Transit VPC

| Page | Title                                          |
|  --- | ---                                            |
|  391 | Transit VPC                                    |
|  398 | global regions with single Transit Hub         |
|  398 | global regions with multiple Transit Hub (GRE) |
|  549 | Direct Connect and Transit VPC                 |

## DX

### Use cases

| Page | Title                                    |
|  --- | ---                                      |
|  591 | VPN as a backup                          |
|  592 | Dual Devices                             |
|  593 | Dual locations                           |
|  594 | Dual locations with DX connection backup |
|  602 | VPN over DX                              |

### Failed cases

| Page | Title                                   |
|  --- | ---                                     |
|  530 | DX Gateway with multiple customer sites |
|  542 | DX Gateway with multiple TGWs           |

### Successful cases

| Page | Title                                   |
| ---  | ---                                     |
| 546  | DX Gateway with multiple customer sites |

## Network Firewall

### Use cases

* Centralised: single firewall subnet / vpc. connect with tgw (791)
* Distributed: firewall subnet per vpc (790)

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
| NAT         | 5-45 Gbps      |
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
