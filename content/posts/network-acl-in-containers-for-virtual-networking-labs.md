---
title: "Network ACL in Containers for Virtual Networking Labs"
date: 2023-05-05
tags:
- docker
- virtual-networking
- acl
---

To set up a virtual networking lab on docker, I need to have the right to modify the network interface.

One way to achieve this is by running a Docker container with the `--cap-add=NET_ADMIN` flag, which grants the container the ability to configure network interfaces. 

Additionally, adding `--net=host` flag to allow the container allow me to access the host network.

```
docker run -it --rm --cap-add=NET_ADMIN --net=host alpine
```

Granting the `NET_ADMIN` capability and using the `--net=host` flag can be potentially dangerous because it allows a container to modify the network configuration of the host system.

However, in a playground or testing environment, these capabilities can be useful for experimenting with different network configurations and scenarios. 

It's important to exercise caution and not use these capabilities in a production environment where security is a top priority.