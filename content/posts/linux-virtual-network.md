---
title: "[Note] Linux Virual Networking"
date: 2023-05-06
tags:
- networking
- bridge
- vlan
- ipvlan
- vxlan
- linux
- docker
---

This post explores various networking concepts and configurations for Linux systems, including: 
- bridging
- VLAN filtering
- IPvlan
-  Vxlan

These concepts are useful for creating and managing virtual networks.

I'm sharing this information based on my own interest and learning experiences. Please note that this may not cover all aspects of networking, and it's recommended to consult additional resources or an expert for more in-depth information.

## Bridge

A bridge connects two or more network segments, allowing them to act as a single network. In Linux, you can create network namespaces and connect them using a bridge. This configuration allows the namespaces to communicate with each other, but they won't have internet access unless you set up NAT on the host.

By default, Docker is using the Bridge network.

[Linux Networking: Bridge, iptables, and Docker](https://aly.arriqaaq.com/linux-networking-bridge-iptables-and-docker/)

A simple lab on it:
- Create two network namespaces, each with its own individual IP address.
- Bridge the namespaces together using the same bridge.
- At this point, the two network namespaces can ping each other, but they won't have internet access.
- To provide internet access, configure NAT on the host machine, which will allow the network namespaces to access the internet.
```
NS1="ns1"
NS2="ns2"

NS1="ns1"
VETH1="veth1"
VPEER1="vpeer1"

NS2="ns2"
VETH2="veth2"
VPEER2="vpeer2"

BR_ADDR="10.10.0.1"
VPEER_ADDR1="10.10.0.10"
VPEER_ADDR2="10.10.0.20"
BR_DEV="br0"

ip netns add $NS1
ip netns add $NS2

ip link add ${VETH1} type veth peer name ${VPEER1}
ip link add ${VETH2} type veth peer name ${VPEER2}

ip link add ${BR_DEV} type bridge
ip link set ${VETH1} master ${BR_DEV}
ip link set ${VETH2} master ${BR_DEV}
ip addr add ${BR_ADDR}/16 dev ${BR_DEV}

ip link set ${VPEER1} netns ${NS1}
ip link set ${VPEER2} netns ${NS2}

ip link set ${BR_DEV} up
ip link set ${VETH1} up
ip link set ${VETH2} up
ip netns exec ${NS1} ip link set lo up
ip netns exec ${NS2} ip link set lo up
ip netns exec ${NS1} ip link set ${VPEER1} up
ip netns exec ${NS2} ip link set ${VPEER2} up
ip netns exec ${NS1} ip addr add ${VPEER_ADDR1}/16 dev ${VPEER1}
ip netns exec ${NS2} ip addr add ${VPEER_ADDR2}/16 dev ${VPEER2}
ip netns exec ${NS1} ip route add default via ${BR_ADDR}
ip netns exec ${NS2} ip route add default via ${BR_ADDR}

bash -c 'echo 1 > /proc/sys/net/ipv4/ip_forward'
iptables -t nat -A POSTROUTING -s ${BR_ADDR}/16 ! -o ${BR_DEV} -j MASQUERADE
```

## VLAN filtering

VLAN filtering allows a single bridge to manage traffic for multiple VLANs, similar to a switch. This enables you to create a single bridge and separate network namespaces into different VLANs, preventing them from communicating with each other.

- Without VLAN filtering: [VLAN Filter Support on Bridge (Without VLAN Filtering)](https://developers.redhat.com/blog/2017/09/14/vlan-filter-support-on-bridge#without_vlan_filtering)
- With VLAN filtering: [Introduction to Linux Bridging Commands and Features (VLAN Filter)](https://developers.redhat.com/articles/2022/04/06/introduction-linux-bridging-commands-and-features#vlan_filter)

## IPvlan

IPvlan allows a single MAC address to have multiple IPs. This can be useful in Docker, where you might want a container to have an IP address in the same subnet as the host network.

[IPvlan Linux Networking](https://www.kernel.org/doc/html/v5.10/networking/ipvlan.html)

```
ip netns add ns0
ip link add link enp0s8 ipvl0 type ipvlan mode l2
ip link set dev ipvl0 netns ns0

ip netns exec ns0 bash
ip link set dev ipvl0 up
ip link set dev lo up
ip addr add 192.168.0.16/24 dev ipvl0
ip addr add 127.0.0.1 dev lo
ip route add default via 192.168.0.1 dev ipvl0
```

## Vxlan

Vxlan is an overlay network that can connect multiple Linux machines and build a network on top of it. This is used in systems like Kubernetes, where the node network is an underlay, and the pod network is an overlay. From the pod's perspective, the node does not exist, and the overlay network appears as a single entity.

[Virtual Networking Labs: Overlay Networks](https://leftasexercise.com/2020/01/03/virtual-networking-labs-overlay-networks/)
