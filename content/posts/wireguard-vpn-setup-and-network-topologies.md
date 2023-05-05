---
title: "WireGuard VPN Setup and Network Topologies"
date: 2023-05-05
tags:
- wireguard
- vpn
- nat
- networking
---

In this blog post, we will explore setting up WireGuard, a modern and secure VPN solution, as well as understanding the NAT approach and various network topologies used in VPN configurations.

## Setting Up WireGuard

Although the official WireGuard website doesn't provide a configuration reference or examples, Stavros' blog post offers a clear guide on configuring WireGuard: [How to Configure WireGuard](https://www.stavros.io/posts/how-to-configure-wireguard/). The WireGuard website does have a quickstart guide, but it focuses on command line setup: [WireGuard Quickstart](https://www.wireguard.com/quickstart/).

## with NAT

For a lab setup using Docker, refer to this repository: [docker-compose.yml](https://github.com/hugotkk/labs/blob/main/wireguard/docker-compose.yml).

The lab setup contains two networks: 
- internet (public subnet)
- hugo_home (private subnet)

The "server" and "client" containers face the public network, with the VPN set up between them. 

The goal is for the client to access services in the private subnet (e.g., "nginx", "httpd", "httpbin") using NAT.

## More use cases

Three topologies are discussed in this article: [WireGuard Point-to-Site Routing](https://www.procustodibus.com/blog/2021/04/wireguard-point-to-site-routing/#masquerading). 

These topologies resemble virtual networking architectures rather than VPN.

### Site-to-Site

To set up a site-to-site connection with WireGuard:

1. Configure services behind SiteA and SiteB endpoints to use their respective endpoints as routers.
2. On the router, add a rule to route traffic going to the other site's prefix to the router at the other site's endpoint.

### Port Forwarding

1. Configure SiteA's endpoint to access services behind SiteB's endpoint, as explained in [Summary of Setting Up Firewall on Different Distros](/posts/summary-of-setting-up-firewall-on-different-distros/).

### Point-to-Site (Site Gateway)

1. Configure SiteA's endpoint to use SiteB's endpoint as the router.
2. SiteB's endpoint will configure a route to direct traffic from SiteA's endpoint prefix to SiteA's endpoint directly.

By following these guidelines, we can set up WireGuard VPN, understand the NAT approach, and apply various network topologies for needs.
