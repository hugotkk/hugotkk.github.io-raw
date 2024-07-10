---
title: Azure Networking
date: 2024-07-03
tags:
  - azure
  - networking
---

## Network Interface

- A single NIC can have multiple public and private IPs.

## Network Security Group (NSG)

NSG is stateful and contains `inbound` and `outbound` rules.

Service Tags are named groups that refer to Azure services (e.g., AzurePortal, AzureLoadBalancer).

An NSG can be attached to:
  - Subnets
  - Network Interfaces

Order of NSG rules check:
  - Inbound: Subnet > NIC
  - Outbound: NIC > Subnet

A single NSG can be attached to multiple subnets or network interfaces.

By default, NSG allows:
- All traffic within the VNET.
- Traffic from the load balancer.

Additional points:
- Once traffic is denied, it is dropped.
- If NSG is attached to both subnet and NIC, traffic must be allowed in both NSGs.
- NSG must be in the same region as the VNET.

To Check NSG for SSH Connectivity (e.g., Can VM1 SSH into VM2?)
1. Identify Source and Destination:
   - VM1 is the source.
   - VM2 is the destination.
2. Check NSG Rules:
   - For the source (VM1), look at the outbound rules.
   - For the destination (VM2), look at the inbound rules.

## Route Table

Imagine we have VNETs A, B, and C that are peered together. VNET C acts as a network VNET to firewall, guard, and audit the network traffic. We want any egress public traffic from VNETs A and B to route through VNET C first.

To achieve this, we can add a route rule in the route tables of VNETs A and B with the address prefix `0.0.0.0/0`, next hop type set to `appliance`, and the next hop address set to the `appliance's address`.

If we want to route all VPN traffic through VNET C, we can set a route table rule at the `GatewaySubnet`.

## Subnet

A subnet can have multiple address spaces, and additional address spaces can be added after the subnet has been created.

Azure reserves the first four and last IP addresses (5 in total) in each subnet.

The smallest subnet size in a VNet is /29.

## Diagnose & Monitor Tools

- Azure Activity Log is equivalent to AWS CloudTrail.
- Azure Advisor: Provides recommendations for cost savings and best practices.

### Dashboard
- Traffic Analytics: showing statistics, summary, numbers of the network

### Diagnose NSG issue
- IP Flow Verify (VM)
- NSG Diagnostics (ScaleSet, NIC, Application Gateway)

### Diagnose Connectivity issue
- Connection Troubleshoot (VMs, Bastion, and Application Gateway)

### Connection monitor

Connection Monitor is a regional service available in each Azure region. 

It continuously checks connections over time.

To utilize Connection Monitor:

- **Azure VMs**: Install the Network Watcher Agent.
- **On-premises**: Install the Azure Monitor Agent. Additionally, use the Azure Connected Machine agent to connect the VMs on Azure.

This setup allows for proactive monitoring of connections, ensuring continuous visibility and troubleshooting capabilities across Azure environments and on-premises infrastructure.

### Flow Log

Enabling Flow Log requires
- Storage Account
- Register Insight Provider (At Subscription)

Enabling Traffic Analytics requires
- Flow Log
- Log Analytics Workspace

### Private Link in Azure Monitor

"Monitor Private Link Scope" refers to a collection of private links.

To use a private connection in Azure Monitor, it is necessary to establish private link connections because Azure Monitor integrates with various services. The "Monitor Private Link Scope" is essential for this purpose.

### Metrics / Insight in Azure Monitor

Azure Monitor is similar to AWS CloudWatch.

To setup the monitoring, we need to

- Install Azure Monitor Agent for collect metrics and logs.
- Setup Data Collection Rule (DCR) to send metrics and logs from the agent to Log Analytics Workspaces.

Components of a DCR:
- **Resources:** Specify VMs from which to collect data.
- **Data Sources:** Include Performance Data, Syslog, and Event Logs gathered by the agent.
- **Destination:** Log Analytics Workspace.

Notice that:
- One DCR can collect data from multiple VMs
- One VM can contribute data to multiple DCRs

This setup allows centralized management and analysis of metrics and logs from Azure VMs using Azure Monitor and Log Analytics Workspaces.

**Legacy Way will use Log Analytics:**
1. Create a Log Analytics workspace.
2. Install the Log Analytics agent.
3. Under the Log Analytics workspace, configure performance counters.

### Monitoring Agents Overview

**Monitor Agent:**
- Capable of sending logs and metrics to multiple Log Analytics Workspaces.
- Recommended as a replacement for other agents.
- Does not support custom logs or IIS logs.

**Windows/Linux Diagnostics Agent:**
- Found in the `Diagnostics` tab.
- Can send logs and metrics to multiple destinations such as Log Analytics Workspaces, Storage Accounts, and Event Hubs.
- Provides more robust functionality compared to Monitor Agent.

Both agents support metrics and logs and are designed for use with VMs.

**Log Analytics Agent:**
- Supports sending logs only.
- Configuration is decentralized (needs to be configured on the VM itself).
- Can be used outside of Azure environments.

## Alerts

### ITSM Integration

One of the alert destinations is send the alert to ITSM via IT Service Management Connector.

But this will be retired in 2025. Currently, it can only be created via API.

### Alert Rate Limits

- **SMS/Voice Call:** Every 5 minutes.
- **Email:** Less than 100 per hour.

Emails will only be sent to individual users, not to groups or service principals.

### Administrative Operations Alerts

You can configure alerts for administrative operations logged in Azure Activity, which essentially covers any API call. Examples include:
- Writing a tag to resource and VM
- Creating a disk
- Attaching a disk to VM

In the activity log, these operations are identified by `"eventSource": "Administrative"`.

### Suppressed Notifications

When an alert processing rule suppresses notifications, the alert will still be listed in the portal, but no notification will be sent.

### Alert Components

- **Resource (Scope):** The subscription or resource the alert pertains to.
- **Condition (Signal):** The source of data and the threshold that triggers the alert.
- **Signal Source:** Typically a Log Analytics Workspace.
- **Action Group:** Defines the notifications and actions to be taken when the alert is triggered.

### User Response States

- If the user response is "New", it can be changed to "Acknowledged" or "Closed."
- If the user response is "Closed", it cannot be changed.


## App Service

Integrating with a VNET provides egress connectivity to the VNET. 

App Service will be assigned a private IP, which it uses to access services within the VNET. 

However, this assignment is dynamic and can change during scaling operations.

Inbound traffic and internet traffic will still use the public IP.

NSGs / Firewall can be utilized to control the App Service traffic flow into the VNET.

## Azure Load Balancer

The SKU of the Load Balancer must match that of the public IP:

- Standard public IP addresses must be used with Standard Load Balancers.
- Basic public IP addresses are compatible only with scale sets or availability sets.

To create the load balancer, a `frontend IP` is required. 

To set up the load balancing rule, a `backend pool` and a `health probe` need to be configured.

The load balancer and VMs need to be in the same VNET.

### NSG

Traffic originating from the LB always appears to come from 168.63.129.16.

When VMs are behind the LB, NSG rules must explicitly specify the source as 168.63.129.16 to allow traffic.

By default, traffic from 168.63.129.16 (Serivce Tag: AzureLoadBalancer) is allowed, so NSG rules are typically used to deny traffic.

### NAT Rules

Load Balancer are used for distributing traffic. Interestingly, in Azure, they can also act as a NAT.

**Inbound NAT Rule:**
- Allows the outside network to reach the backend point or VMs behind the backend pool
- To connect to VM1 and VM2 via the Load Balancer on same port, we need 2 frontend IPs:
  - `<frontend_ip1>:8080 -> VM1:8080`
  - `<frontend_ip2>:8080 -> VM2:8080`
- Frontend IPs can be mapped to a backend pool using a single frontend IP, where the frontend IP starts with a base port that increments for each mapping:
  - `<frontend_ip1>:8081 -> VM1:8080`
  - `<frontend_ip1>:8082 -> VM2:8080`
  - `<frontend_ip1>:X -> VMX:8080`

**Outbound Rules:**
- Multiple VMs in the backend pool can share a single frontend IP for outbound traffic.

### Floating IP

- **Disabled:** Traffic routes through the load balancer to the VM's IP.
- **Enabled:** Traffic routes through the load balancer to the VM's Floating IP (FIP).

In an on-premise Active-Passive HA setup, a VIP is typically used, to which clients always connect. Monitoring tools check server health and shift the VIP between active and passive instances. The VM will respond to ARP requests for the VIP when it holds it.

In the cloud, ARP and other broadcast messages are not permitted, necessitating the use of a load balancer.

To set up a Floating IP in Linux:
- Basically, create a loopback interface with the Floating IP.

This setup is commonly used to achieve High Availability in SQL Server, where a Floating IP is needed for direct server return.

### HA Port

Configuration: Set port to 0 and protocol to all in the load balancing rule. This directs all ports and protocols to the VM directly

Due to this, each floating IP can only have 1 load balancer rule and connect to 1 backend.

This feature is for:
- **Internal Load Balancer** only
- **Standard tier** only

**Use Case: Virtual Appliance (eg: Firewall):**
- Configure two floating IPs and HA ports on the virtual appliance to tag traffic from different services.
- HA port is also required to inspect traffic for any protocol and any port.
- Floating IP is necessary for HA and to distinguish the traffic from different service, such as:
  - Route all Web Server subnet traffic to FIP1.
  - Route all DB subnet traffic to FIP2.
- Requests are first forwarded to the virtual appliance (acting as a man in the middle) before being sent to the destination.

### Session Persistence 

Session Persistence based on Client IP ensures that traffic from the same client is always served by the same VM.

## VM

A VM without a public IP can still access the internet.

For a VM with standard public IP:
- Security settings enforce blocking all traffic by default. You must attach a NSG to allow traffic into the VM.

For a VM with basic public IP:
- The VM is accessible publicly without needing an NSG attached.

A VM can change its subnet or NIC but not its VNET. To change the VNET or move the VM to another region, we need to recreate the VM.

## Bastion

- Bastion must be deployed to a subnet named AzureBastionSubnet.
- The subnet must be /26 or larger.
- Bastion protects VMs only.

- Basic SKU: 
  - Creates 2 instances.
- Standard SKU: 
  - Allows specifying the number of instances (host scaling).
  - Supports file upload/download.
  - Supports native client (use Azure CLI for SSH/RDP connections).

Public IP requirements:
- Can be regional or global (to be confirmed).
- Must be static.
- Requires Standard SKU.

Without the native client, we can only use Bastion through the Azure portal.

Each instance supports up to 20 RDP or 40 SSH connections.

For example, with 100 concurrent SSH connections, we need 3 instances, which means we need the Standard Bastion SKU and host scaling set to 3. 

Resizing the subnet is unnecessary since we only need additional instances.

## Expressroute

- The Basic SKU does not support the coexistence of VPN Gateway and ExpressRoute.
- The ErGw3Az SKU supports FastPath.

## VNET Peering

With VNET Peering, the VNET address spaces must not overlap.

When peering is disconnected, we need to recreate the peering connections.

When transit routing is enabled in a VNET, traffic can transit through it to reach other VNETs.

For example, if we have VNET1, VNET2, and VNET3:
- Initially, VNET1 cannot access VNET3.
- If transit routing is enabled in VNET2, VNET1 can access VNET3 through VNET2.

One key difference between S2S VPN and VNET Peering is that VNET Peering does not require creating an extra GatewaySubnet to connect two VNETs.

## Firewall

Firewall Premium supports public IP with:
- Standard SKU
- Global & Regional? (to be confirmed)
- IPv4 Only

The region of the firewall should be the same as the region of the VNET.

In App Service, to control outbound traffic with a firewall, we need to enable VNET integration and then create a user-defined route table to route all traffic to the firewall.

## Service Endpoint

Service Endpoints use private IP addresses to access Azure services, simplifying access control with ACLs.

VNET and Service Endpoint must be in the same region.

It is in the Subnet level.

## VirtualWAN

Virtual WAN (similar to AWS Transit Gateway) is a global resource that can span across regions, while Virtual Hubs are regional resources. 

A single Virtual WAN can connect to multiple Virtual Hubs.

To create a S2S VPN between a branch and a VNET:

1. Create a Virtual WAN.
2. Create a Virtual Hub.
3. Create a VPN site in the Virtual Hub.
4. Connect the VPN site to the Hub.
5. Connect the VNET to the Hub.

## VPN

### S2S VPN

The Basic SKU VPN Gateway does not support coexistence with ExpressRoute for S2S VPN.

**Steps to Create a S2S VPN**
1. Create a gateway subnet.
2. Create a VPN gateway.
3. Create a local gateway (representing the on-premise site).
4. Create a VPN connection.

Each VPN gateway can support 2 connections, requiring 2 local gateways (for 2 on-premise routers) and using up 2 public IP addresses for the gateway.

- **IKEv2** supports up to 10 S2S connections.
- **IKEv1** supports only 1 S2S connection.

IKEv2 is necessary if redundancy is required.

### P2S VPN

#### When to Use

- **P2S VPN:** Suitable for remote workers, as the IP addresses connecting to the VNET can vary.
- **S2S VPN:** Suitable for branch offices, connecting on-premise networks to the VNET with fixed IPs.

#### Authentication

P2S VPN can authenticate using 
- certificate authentication
- Active Directory

**To setup Certificate Authentication:**
1. Generate a CA and client certifcate.
2. Upload the CA public key to the P2S VPN Setting.
3. Connect to the VPN using client certifcate.

#### Common issue

1. The on-premise network is connected to VNET A via a S2S VPN.
2. Clients connect to VNET A using a P2S VPN.
3. VNET A is peered with VNET B.

P2S VPN clients may fail to access VNET B when the network topology changes, necessitating a reinstall or restart of the VPN client. 

In this setup, the client cannot access VNET B, while the on-premise network can, indicating this issue.

### VPN Types

- **Route-based VPN:** Used for large networks; uses a route table.
- **Policy-based VPN:** Used for small networks; uses ACLs, requiring specific source and destination addresses.

Since usually P2S VPN clients do not have fixed IPs, it is difficult to use ACLs (as the source cannot be specified). Therefore, route-based VPN is usually used for P2S VPNs.
Certainly! Here's a rewritten version:

## DNS

DNS settings can be configured at the 
- VNET
- NIC

### Custom Domains

1. Update NS records at your domain registrar.
2. Verify domain ownership using a TXT record.
3. Configure your custom domain with a CNAME or A record.

### Zone Migration

Azure offers PowerShell scripts to facilitate DNS zone migration, useful for transitioning from on-premises or other cloud providers.

### Private DNS Zones

- Private DNS zones are necessary for connecting with Virtual Networks (VNETs).
- One DNS zone can be connected to multiple VNETs, and multiple resolution zones can be linked to a single VNET.
- Auto-registration automated the creation of DNS records for linked VNETs.
- Auto-registration is only supported within private DNS zones.
- Each VNET can designate one registration zone with auto-registration enabled.

### Custom DNS Server

- Virtual Machines (VMs) can function as custom DNS servers.
- Each VNET can opt for Azure DNS or a custom DNS setup.
- Ensure connectivity to the VM when using it as a DNS server.

### Application Gateway

- similar to AWS's ALB
- Level 7 Application Load Balancer
- Regional Service

### Traffic Manager

- DNS-based traffic routing service
- Global service, not tied to a specific region
- Can handle regional outages by redirecting traffic to healthy endpoints