---
title: "Setting Up a Master-Slave DNS Server with Bind9 on Ubuntu & Docker"
date: 2023-05-05
tags:
- bind9
- dns
---

## Setup guide

With Ubuntu: https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-private-network-dns-server-on-ubuntu-20-04.

With Docker: https://github.com/hugotkk/lab-docker-bind.

## Zone Transfer

With zone transfer enable, when a DNS record is updated in the primary DNS server, the update will be transferred automatically to the secondary DNS server.


In the primary server, we should set

```
type private
allow-transfer { <secondary-ip>; }
```

In the secondary server, we should set 

```
type slave;
masters { <primary-ip> };
```

## Recursive Queries

Also, if we want to enable recursive queries (forwarding queries to a forwarder if the answer isn't in the local DNS), we should only allow trusted servers to do so, especially if we're a private DNS server.

```

acl "trusted" {
    <trusted_servers>;
};

options {
    ....
    recursion yes;
    allow-recursion { trusted; };
    forwarders {
        8.8.8.8;
        8.8.4.4;
    };
```

To accomplish this, create an ACL that includes the IP addresses of trusted servers. 

In the `options` section, 
- enable recursion
- specify the `trusted` ACL in the `allow-recursion`. 
- add one or more forwarders (e.g., public DNS servers such as 8.8.8.8 and 8.8.4.4) in the `forwarders`.