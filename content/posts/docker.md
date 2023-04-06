---
title: "Docker"
date: 2022-12-13
tags:
- docker
---

## Dockerfile

Most containers are not well-documented, making it hard to find essential information about them on Docker Hub or GitHub's README. Some of the critical information to know includes:

* Which user does the container run as?
* Which ports are used by the application?
* Where are the config files/directory?
* Where will the persistent data be stored?

The best way to find these answers is by examining the Dockerfile. For example:

[Prometheus](https://github.com/prometheus/prometheus/blob/main/Dockerfile)

- User: `nobody`
- Port: `9090`
- Config path: `/etc/prometheus/prometheus.yml`
- Data path: `/prometheus`

[php:8-apache-buster](https://github.com/docker-library/php/blob/cc901e9594f79c305d9508e5b5df572eb93245b4/8.2/buster/apache/Dockerfile)

- Port: `80`
- PHP config: `/usr/local/etc/php/php.ini`
- Apache config: `/etc/apache2/conf-available/docker-php.conf`
- Document root: `/var/www/html/`

## Disk Usage

Analyse the disk usage of docker

- Run the command: `docker system df -v`
- Reference: https://docs.docker.com/engine/reference/commandline/system_df/

## Live Restore

This will keep containers running during the docker upgrade and `systemctl stop dockerd`

- Edit the file `/etc/docker/daemon.json`
- Add the following content to the file:

```
{
  "live-restore": true,
}

```

## Address pool

- To override the default address pool, add the following content to the `daemon.json` file:

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

- Reference: https://forums.docker.com/t/docker-default-address-pool-customization-question/112969


## Logging

* Docker logging does not rotate by default.
* To change the logging driver to JSON, you need to modify the Docker daemon configuration file (/etc/docker/daemon.json) and add the following line:
```
{
  "log-driver": "json-file"
}
```
* To set the retention policy for the JSON logging driver, you can add the following configuration options to the same file:
```
{
  "log-opts": { 
    "max-size": "10m", 
    "max-file": "3"
  }
}
```
* This will retain a maximum of 3 log files, each with a maximum size of 10 MB.
* After making changes to the configuration file, you need to restart the Docker daemon for the changes to take effect:
```
sudo systemctl restart docker
```

## Docker Swarm

### Service Management

- To reference a service, use `tasks.<service-name>` instead of a specific replica.
- Publishing a port will expose it on all Swarm nodes, requiring a load balancer to balance the service.
- A load balancer distributes incoming requests to nodes running the service, which will handle the request using one of its replicas.
- Horizontal scaling of the service is achieved by adding additional replicas to handle increased traffic.

### Volume Management

- Volumes/local mount points are independent on each node.
- When creating a volume/local mount point for a service, it is created on the node where the task is assigned.
- Changes made to a volume/local mount point on one node will not be reflected on other nodes.
- To ensure data consistency across nodes, use a distributed file system like GlusterFS or NFS, or a cloud-based storage service like Amazon EFS or Google Cloud Storage.
- When using a distributed file system or cloud-based storage service, the volume/local mount point is shared across all nodes in the Swarm, ensuring that all nodes have access to the same data.

## Useful links

* [install docker](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)
* [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)
* [docker-comopse.yml reference](https://docs.docker.com/compose/compose-file/)
* [/etc/docker/daemon.json reference](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file)
* [how to trust the tls certificate of a registry with TLS enabled](https://docs.docker.com/registry/insecure/#/docker-still-complains-about-the-certificate-when-using-authentication)

