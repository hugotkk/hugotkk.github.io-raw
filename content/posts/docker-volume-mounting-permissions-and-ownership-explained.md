---
title: "Docker Volume Mounting: Permissions and Ownership Explained"
date: 2023-05-04
tags:
- docker
- volume
- mount
---

There are two ways to mount volumes in Docker:

1. Hostpath:
```
docker run -it --rm -v $PWD/myvol:/app ubuntu
```
2. Volume:

```
docker volume create myvol
docker run -it --rm -v myvol:/app ubuntu
```

When dealing with volume ownership, incorrect folder ownership can cause the container to crash, requiring time to debug logs.

Common issues:
- Permission denied errors in logs
- Bitnami images built with non-root user
- OpenShift starts containers with random UID

Examples of volume permissions in Docker:

Prometheus:
- Data store: `/prometheus`
- Config path: `/etc/prometheus`

Rules for volume ownership and permissions:

1. If the host volume doesn't exist, Docker daemon will create it with `root` ownership.

Example:

```
docker run -it --rm -v $PWD/data:/data prometheus
```

`$PWD/data` will be owned by `root`.

2. If the host volume doesn't exist but the path exists in the container, a new volume will be created, and the files inside that path will be copied to it. Ownership will follow the one inside the container.

Example:

```
docker run -it --rm -v $PWD/data:/etc/prometheus prometheus
```

`$PWD/data` ownership will be the same as `/etc/prometheus`. Files in /etc/promethues will be copied as well.

3. If the volume exists, it will follow its own ownership.

Example: (@ubuntu)
```
mkdir data
docker run -it --rm -v $PWD/data:/etc/prometheus prometheus
````

`$PWD/myvol` will be owned by `ubuntu`.

Issues arise when the container is running as a non-root user. In these cases, folders may be writable by the container or the host, but not both.

Solutions for non-root user containers (not ideal):

1. Start the container with `root` user (may not work for applications that require a specific user, e.g., MariaDB).

2. On the host, create the folder first, then use sudo to change the volume owner to the container user.

3. Change the folder permission to `777`, allowing both host and container to write to it.
