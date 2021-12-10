---
title: "EC2 Bandwidth test"
date: 2021-12-10
tags:
- aws
- ec2
- bandwidth
---

# Objective

* Test if the ec2 can reach 10Gbps bandwidth in same placement group
* Test if the 10Gbps is the limited by the ec2 instance type or aws

## AWS Limit

https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-network-bandwidth.html

in single-flow traffic,

* placement group: 10Gbps
* w/o placement group: 5Gbps

flow = worker (thread) in the network driver. depends on the ec2 instance type.

### EC2 Instance Type Limit

https://aws.amazon.com/ec2/instance-types/m5/

m5.8xlarge can reach 10Gbps

# Setup

Created 3 m5.8xlarge instances which can reach 10Gbps bandwidth each (i use spot)

* 2 in same placement group
* 1 in other subnet

## Tool

* iperf ([BENCHMARK NETWORK THROUGHPUT](https://ec2-immersionday.workshop.aws/benchmark-network-throughput.html))

# Test

## Not in same placement group and parallel 1

```
[root@ip-172-31-33-28 ~]# iperf -c 172.31.12.68 --parallel -i 2 -t 2
iperf: ignoring extra argument -- 2
------------------------------------------------------------
Client connecting to 172.31.12.68, TCP port 5001
TCP window size:  812 KByte (default)
------------------------------------------------------------
[  3] local 172.31.33.28 port 38880 connected with 172.31.12.68 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3]  0.0- 2.0 sec  1.15 GBytes  4.93 Gbits/sec
[root@ip-172-31-33-28 ~]# iperf -c 172.31.12.68 --parallel -i 2 -t 2
iperf: ignoring extra argument -- 2
```

## In same placement group and parallel 1

```
[root@ip-172-31-7-236 ~]# iperf -c 172.31.12.68 --parallel -i 2 -t 2
iperf: ignoring extra argument -- 2
------------------------------------------------------------
Client connecting to 172.31.12.68, TCP port 5001
TCP window size: 1.90 MByte (default)
------------------------------------------------------------
[  3] local 172.31.7.236 port 50544 connected with 172.31.12.68 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3]  0.0- 2.0 sec  2.22 GBytes  9.54 Gbits/sec
```

## Not in same placement group and parallel 2

```
[root@ip-172-31-7-236 ~]# iperf -c 172.31.12.68 --parallel 2 -i 1 -t 2
------------------------------------------------------------
Client connecting to 172.31.12.68, TCP port 5001
TCP window size: 2.00 MByte (default)
------------------------------------------------------------
[  4] local 172.31.7.236 port 50538 connected with 172.31.12.68 port 5001
[  3] local 172.31.7.236 port 50536 connected with 172.31.12.68 port 5001
[ ID] Interval       Transfer     Bandwidth
[  4]  0.0- 1.0 sec   603 MBytes  5.06 Gbits/sec
[  3]  0.0- 1.0 sec   585 MBytes  4.91 Gbits/sec
[SUM]  0.0- 1.0 sec  1.16 GBytes  9.96 Gbits/sec
[  4]  1.0- 2.0 sec   604 MBytes  5.07 Gbits/sec
[  4]  0.0- 2.0 sec  1.18 GBytes  5.06 Gbits/sec
[  3]  1.0- 2.0 sec   579 MBytes  4.85 Gbits/sec
[SUM]  1.0- 2.0 sec  1.16 GBytes  9.92 Gbits/sec
[  3]  0.0- 2.0 sec  1.14 GBytes  4.88 Gbits/sec
[SUM]  0.0- 2.0 sec  2.32 GBytes  9.94 Gbits/sec
```

## Outcome

### w/i the placement group

| Flow | Bandwidth/Flow | Bandwidth |
|  --- | ---            | ---       |
|    1 | 10Gbps         | 10Gbps    |
|    2 | 5Gbps          | 10Gbps    |

we can use up all m5.8xlarge's bandwidth in a single-flow traffic. 

But the traffic splits 50%,50% in double-flow traffic. 

Therefore, 10Gbps is the max bandwidth of the m5.8xlarge.

### outside the placement group

| Flow | Bandwidth/Flow | Bandwidth |
|  --- | ---            | ---       |
|    1 | 5Gbps          | 5Gbps     |
|    2 | 5Gbps          | 10Gbps    |

we need two flows to use up all m5.8xlarge's bandwidth (5Gbps, 5Gbps)

so it reaches the limit of AWS...
