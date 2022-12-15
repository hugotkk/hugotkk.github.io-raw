---
title: "Notes about Docker"
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

## Useful links

* [install docker](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)
* [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)
* [docker-comopse.yml reference](https://docs.docker.com/compose/compose-file/)
* [/etc/docker/daemon.json reference](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file)
* [how to trust the tls certificate of a registry with TLS enabled](https://docs.docker.com/registry/insecure/#/docker-still-complains-about-the-certificate-when-using-authentication)

