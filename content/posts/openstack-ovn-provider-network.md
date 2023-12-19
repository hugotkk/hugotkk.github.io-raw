---
title: "Openstack - OVN - Provider Network"
date: 2023-12-19
tags:
- openstack
- networking
- ovn
---

## Concept

<img src="https://res.cloudinary.com/canonical/image/fetch/f_auto,q_auto,fl_sanitize,c_fill,w_720/https://lh4.googleusercontent.com/QpDuuauuQ5jitdLb9h8KahX0W_ex_T2yCG1DMb60vcnWloAj5ecm0B_rJQUCpAN63Eql2aiUp7O7ipPB1UrNZnxi_hEam6vu4jcGSXmMlK3mGB-xCl__KhQCT-g7vd9LTc3VjH1Y"/>

https://ubuntu.com/blog/data-centre-networking-what-is-ovn

OVN manages the Northbound and Southbound DB, playing a crucial role in network orchestration:

**Northbound DB**
- Functions as a public API service, facilitating interactions with external services, such as the OpenStack API.

**Southbound DB**
- Manages OVN's internal data, allowing for communication among OVN controllers.

**Northbound to Southbound Translation (northd):**
- Operates as an intermediary, converting instructions from the Northbound DB to a format understandable by the Southbound DB.

**Cli:**
- `ovn-nbctl`: Manages the definitions of routers, bridges, and networks within the cluster.
- `ovn-sbctl`: Manages the information related to servers.

**Compute Node Network Integration:**
- Each compute node runs an OVN controller that connects to the centralized Southbound OVN DB located on the network node.
- This OVN controller coordinates networking operations by communicating with a local instance of OVS server.

The local database can be managed using the `ovs-vsctl` command.

Two crucial parameters in the networking context are:

1. `ovn-bridge-mappings`: This parameter establishes mappings between OVN logical networks and OVS bridges.

2. `enable-chassis-as-gw`: It configures the chassis to act as gateways when the need arises.

## Troubleshooting

### Misconfig in provider network

When setting up a provider network in Horizon, the network config must be specified as a `flat` and `external` type. The `physical network` designation corresponds to the identifier used within the OVS DB.

For instance, consider we have a provider network `public` and a corresponding physical network named `providernet`.

In such a scenario, the details of the network and its associated ports can be observed within the Northbound DB.

```bash
ovn-nbctl list logical_switch public
```

```
_uuid               : d3cb813b-c44b-4782-87bb-aa6593d40357
acls                : []
copp                : []
dns_records         : []
external_ids        : {"neutron:availability_zone_hints"="", "neutron:mtu"="1500", "neutron:network_name"=public, "neutron:revision_number"="1"}
forwarding_groups   : []
load_balancer       : []
load_balancer_group : []
name                : neutron-f5f8f47b-4093-42ae-a5e9-431ef06e5eeb
other_config        : {mcast_flood_unregistered="false", mcast_snoop="false", vlan-passthru="false"}
ports               : [06684ced-8c82-400c-9bc8-5f1ba35b3687, 439f0fe3-fb00-4679-b7ca-0a850e4b6cbb, d38818c3-0ba5-4d82-8a88-b2fcfdc29a70]
qos_rules           : []
```

d38818c3-0ba5-4d82-8a88-b2fcfdc29a70 is the port of the provider network

```
ovn-nbctl list logical_switch_port d38818c3-0ba5-4d82-8a88-b2fcfdc29a70
```

```
_uuid               : d38818c3-0ba5-4d82-8a88-b2fcfdc29a70
addresses           : [unknown]
dhcpv4_options      : []
dhcpv6_options      : []
dynamic_addresses   : []
enabled             : []
external_ids        : {}
ha_chassis_group    : []
name                : provnet-840f094f-44df-4746-bdfe-8007ec2dd64f
options             : {mcast_flood="false", mcast_flood_reports="true", network_name=providernet}
parent_name         : []
port_security       : []
tag                 : []
tag_request         : []
type                : localnet
up                  : false
```

In the OVS DB, the `ovn-bridge-mappings` config links the physical network named `providernet` to the bridge device `br-ex`.

It is essential that the information in the OVS DB matches with the data in the Northbound DB.

```bash
ovs-vsctl list open
```

```
_uuid               : bc26d1c9-8134-4550-8099-9ebc34cd3509
bridges             : [598efdc0-15f6-4dcb-924f-80c6bad3c1d3, ade90deb-b74e-4eaf-97f0-d56309aadac4]
cur_cfg             : 4
datapath_types      : [netdev, system]
datapaths           : {system=d4540bf0-d630-448c-9fee-98d051f247e5}
db_version          : "8.3.0"
dpdk_initialized    : false
dpdk_version        : none
external_ids        : {hostname=ubuntu-jammy, ovn-bridge=br-int, ovn-bridge-mappings="providernet:br-ex", ovn-cms-options=enable-chassis-as-gw, ovn-encap-ip="10.0.114.11", ovn-encap-type=geneve, ovn-remote="tcp:10.0.114.11:6642", rundir="/var/run/openvswitch", system-id="b13926a9-19ed-46c8-a877-3df39d32782b"}
iface_types         : [bareudp, erspan, geneve, gre, gtpu, internal, ip6erspan, ip6gre, lisp, patch, stt, system, tap, vxlan]
manager_options     : [0966f00e-7387-432a-a534-e3d1831b809e]
next_cfg            : 4
other_config        : {vlan-limit="0"}
ovs_version         : "2.17.7"
ssl                 : []
statistics          : {}
system_type         : ubuntu
system_version      : "22.04"

```

If the mappings are not correctly configured, the `ovs-vsctl show` command will display only two bridges: `br-int` and `br-ex`.

`br-ex` will be associated with the network device `enp0s9`, but without the proper linkage, there will be no connection between `br-int` and `br-ex`. 

This means that the bridges will operate independently and not pass traffic between each other.

```bash
ovs-vsctl show
```

```
bc26d1c9-8134-4550-8099-9ebc34cd3509
    Manager "ptcp:6640:127.0.0.1"
        is_connected: true
    Bridge br-int
        fail_mode: secure
        datapath_type: system
        Port br-int
            Interface br-int
                type: internal
    Bridge br-ex
        Port enp0s9
            Interface enp0s9
        Port br-ex
            Interface br-ex
                type: internal
    ovs_version: "2.17.7"

```

An automatic creation of a `patch-XXX` port occurs when a corresponding mapping is found. This implies that the provider network must be operational.

```bash
ovs-vsctl show
```

```
bc26d1c9-8134-4550-8099-9ebc34cd3509
    Manager "ptcp:6640:127.0.0.1"
        is_connected: true
    Bridge br-int
        fail_mode: secure
        datapath_type: system
        Port br-int
            Interface br-int
                type: internal
        Port patch-br-int-to-provnet-9c84ee84-3b4d-4dc8-8317-1f0939603413
            Interface patch-br-int-to-provnet-9c84ee84-3b4d-4dc8-8317-1f0939603413
                type: patch
                options: {peer=patch-provnet-9c84ee84-3b4d-4dc8-8317-1f0939603413-to-br-int}
    Bridge br-ex
        Port patch-provnet-9c84ee84-3b4d-4dc8-8317-1f0939603413-to-br-int
            Interface patch-provnet-9c84ee84-3b4d-4dc8-8317-1f0939603413-to-br-int
                type: patch
                options: {peer=patch-br-int-to-provnet-9c84ee84-3b4d-4dc8-8317-1f0939603413}
        Port enp0s9
            Interface enp0s9
        Port br-ex
            Interface br-ex
                type: internal
    ovs_version: "2.17.7"
```

Incorrect setup will result in the VM lacking internet connectivity. 

While it's still possible to attach a floating IP to the VM, it won't work as expected. 

The VM won't be able to ping essential destinations like 8.8.8.8, the gateway, or any other machines within the provider network.

### Broken gateway

Once the bridge mapping is corrected, all ports will appear as active in OpenStack admin. 

However, internet access will cease when the compute node is offline. In this scenario, the VM resides on the controller node:

- Controller: Up (VM is located here)
- Compute: Down

```bash
ovn-nbctl show
```

```
...
router 680f21a0-2a4c-49e1-962e-869c33bdca08 (neutron-5351b89f-04d1-4498-9a03-6d5db3b016a2) (aka router1)
    port lrp-ca9fc58e-5a6e-431e-b7d9-b310413be9bf
        mac: "fa:16:3e:56:31:9b"
        networks: ["10.0.0.1/22"]
    port lrp-b08ccb1f-5481-430e-9bff-8a4623924186
        mac: "fa:16:3e:81:6d:4d"
        networks: ["203.0.114.124/24"]
        gateway chassis: [f83ca915-8312-42dd-9f7c-3b7f6e128a16 b612c66f-6b89-4e08-9c8a-2b8da2037a60]
    nat 0f48c87b-9173-46f4-a6a2-8d90c8e8a779
        external ip: "203.0.114.124"
        logical ip: "10.0.0.0/22"
        type: "snat"

```

```bash
ovn-sbctl show
```

```
Chassis "b612c66f-6b89-4e08-9c8a-2b8da2037a60"
    hostname: ubuntu-jammy
    Encap geneve
        ip: "10.0.114.11"
        options: {csum="true"}
Chassis "f83ca915-8312-42dd-9f7c-3b7f6e128a16"
    hostname: compute
    Encap geneve
        ip: "10.0.114.12"
        options: {csum="true"}

```

The gateway chassis contains two entries: 
- controller
- compute (f83ca915-8312-42dd-9f7c-3b7f6e128a16)

Since the compute was offline, it was the root cause of the network connectivity problem.

To address this issue initially, I removed the compute from the gateway chassis, which resolved the network connectivity problem.

```bash
ovn-nbctl lrp-del-gateway-chassis lrp-b08ccb1f-5481-430e-9bff-8a4623924186 f83ca915-8312-42dd-9f7c-3b7f6e128a16
```

However, the problem recurred shortly after the config was corrected. This prompted an investigation into why the compute node kept being designated as a gateway chassis.

Upon closer examination, it became evident that the issue originated within the compute node itself. 

Running the command `ovs-vsctl list open` revealed a config entry labeled `ovn-cms-options=enable-chassis-as-gw`.

When the `ovn-cms-options=enable-chassis-as-gw` config is present, the compute node is automatically assigned as a gateway chassis for the router.

The root cause of this recurrent behavior was found in the DevStack script, where the default setting for `ENABLE_CHASSIS_AS_GW` is configured to be true, leading to the compute node repeatedly assuming the role of a gateway chassis.

**Updated as of 2023-10-05**:

The assignment of the gateway chassis should exclusively occur on dedicated network nodes.

Within OVN, there exists a concept known as Distributed Floating IP, which operates as follows:

1. For VMs lacking a Floating IP, external traffic is routed through the network Node.

2. When a Floating IP is attached, traffic is directed through the local bridge for direct internet access.

3. The L3 High Availability (L3HA) support feature is implemented to initiate a failover in the event of node failure.

In contrast, in an OVS setup, traffic consistently passes through the local bridge.

Reference:

OVN: <https://docs.openstack.org/networking-ovn/latest/admin/routing.html>

OVS: <https://docs.openstack.org/neutron/2023.1/admin/deploy-ovs-provider.html#deploy-ovs-provider>