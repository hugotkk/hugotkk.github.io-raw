---
title: "An Introduction to Pacemaker: Ensuring High Availability and Auto-Failover for Your Cluster"
date: 2023-05-05
tags:
- pacemaker
- ha
---

Pacemaker is a powerful cluster resource manager designed to provide high availability (HA) and auto-failover capabilities for services and applications. In this blog post, we will cover some essential aspects of Pacemaker, from its basic functionality to different modes and installation.

Key Points to Know About Pacemaker:

1. Role in a Cluster:
Pacemaker works like a 'systemctl' for clusters, ensuring that services remain available and automatically failing over to another node if a primary node goes down.

2. Default Behavior:
By default, Pacemaker maintains a single running service in the cluster and adds the service to other nodes when the primary node is down.

3. Operating Modes:
Pacemaker offers various modes, including clone mode and master-slave mode, to suit different cluster configurations and requirements.

4. Auto-Failover:
Auto-failover can be set up using a virtual IP (VIP) and configuring constraints for the 'IPaddr2' resource to follow the master node.

5. Fencing:
Pacemaker enables fencing by default to isolate and protect resources. However, it can be disabled by running the command `pcs property set stonith-enabled=false`.

6. Two-Node Clusters:
For two-node clusters, it is necessary to disable quorum with the command `pcs property set no-quorum-policy=ignore`.

7. Pacemaker Web GUI:
Pacemaker's Web GUI is a fully functional command line interface that allows you to manage all aspects of the cluster.

8. Galera and Master-Slave Mode:
Galera is a special case of master-slave mode, where it operates as a multi-master configuration. To set up Galera, you need to:
  - Set `master-max = <no_of_node>`
  - Manually boot the cluster to generate `grastate.dat`

9. Installing Pacemaker:
To install Pacemaker, refer to the installation guide provided by ClusterLabs [here](https://clusterlabs.org/quickstart-redhat.html).

Pacemaker is a robust and flexible cluster resource manager that provides essential high availability and auto-failover capabilities for various services and applications. Understanding its core features and configuration options can help ensure that your cluster remains operational and resilient to failures.
