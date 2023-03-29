---
title: "Docker"
date: 2022-12-13
tags:
- docker
---

## Self-document from Dockerfile

Most of the containers are not very well-documented. It makes it hard to find out the information about the container from docker hub and github's README.

Usually I would like to know:
* Which user does this container run as?
* Which ports are used by the application?
* Where are the config files / directory?
* Where will the persistent data be stored?

The best way to find those answers is to look at the Dockerfile.

By looking at the [Dockefile](https://github.com/prometheus/prometheus/blob/main/Dockerfile) of prometheus, we would know:

* user: nobody
* port: 9090
* config path: /etc/prometheus/prometheus.yml
* data path: /prometheus

For [php:8-apache-buster](https://github.com/docker-library/php/blob/cc901e9594f79c305d9508e5b5df572eb93245b4/8.2/buster/apache/Dockerfile),

* port: 80
* php config: /usr/local/etc/php/php.ini
* apache config: /etc/apache2/conf-available/docker-php.conf
* document root: /var/www/html/

## Analyse the disk usage of docker

```
docker system df -v
```

https://docs.docker.com/engine/reference/commandline/system_df/

## Keep containers running during the docker upgrade / systemctl stop dockerd

```
vi /etc/docker/daemon.json
```

```
{
"live-restore": true,
}

```


## Change default address pool

The address pool may conflict with the internal network. We can override it via daemon.json

```
{
  "default-address-pools": [
    {
      "base": "10.1.0.0/16",
      "size": 24
    }
  ]
}
```

https://forums.docker.com/t/docker-default-address-pool-customization-question/112969

## logging

* Docker logging does not rotate by default.
* To change the logging driver to JSON, you need to modify the Docker daemon configuration file (/etc/docker/daemon.json) and add the following line: "log-driver": "json-file"
    * To set the retention policy for the JSON logging driver, you can add the following configuration options to the same file:
    * "log-opts": { "max-size": "10m", "max-file": "3" }
    * This will retain a maximum of 3 log files, each with a maximum size of 10 MB.
    * After making changes to the configuration file, you need to restart the Docker daemon for the changes to take effect:
    * sudo systemctl restart docker

# Docker Swarm

* A container cannot reference a specific replica, but you can use tasks.<service-name> to reference the service.
* When publishing a port, it will be exposed on each Swarm node, and you can use a load balancer to balance the service.
* The load balancer will distribute incoming requests to the nodes running the service, and the service will handle the request using one of its replicas.
* This allows for horizontal scaling of the service, where additional replicas can be added to handle increased traffic.

* Volumes/local mount points are independent in each node.
* When you create a volume/local mount point for a service, it is created on the node where the task is assigned.
* This means that each node will have its own copy of the volume/local mount point, and changes made to it on one node will not be reflected on other nodes.
* To ensure data consistency across nodes, you can use a distributed file system like GlusterFS or NFS, or a cloud-based storage service like Amazon EFS or Google Cloud Storage.
* When using a distributed file system or cloud-based storage service, the volume/local mount point is shared across all nodes in the Swarm, ensuring that all nodes have access to the same data.

## Useful links

* [install docker](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)
* [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)
* [docker-comopse.yml reference](https://docs.docker.com/compose/compose-file/)
* [/etc/docker/daemon.json reference](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file)
* [how to trust the tls certificate of a registry with TLS enabled](https://docs.docker.com/registry/insecure/#/docker-still-complains-about-the-certificate-when-using-authentication)

