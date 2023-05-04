---
title: "Networking in Docker and Kubernetes: Port Expose and Pod Communication"
date: 2023-05-04
tags:
- k8s
- docker
---

# Docker

```
ubuntu@ip-172-31-6-200:~$ docker run --rm -it -P httpd
```

```
ubuntu@ip-172-31-6-200:~$ docker ps 
CONTAINER ID   IMAGE     COMMAND              CREATED          STATUS          PORTS                                     NAMES
6f6f53ee7dde   httpd     "httpd-foreground"   12 seconds ago   Up 10 seconds   0.0.0.0:32768->80/tcp, :::32768->80/tcp   dreamy_goodall
```

- Docker automatically knows which ports a container needs due to the `EXPOSE` directive in the container's Dockerfile. (httpd's [Dockerfile](https://github.com/docker-library/httpd/blob/master/2.4/Dockerfile))
- Removing `EXPOSE` from the Dockerfile will not publish the port automatically anymore when using the -P flag.
- Howerver, containers can still reach ports within the container via its IP/alias.
- Docker does not enforce any security policies between containers.

We can test it by:
- Create a custom network
- Start two containers, connected to the same network
- Install net-tools on both containers
- Listen to port 3000 on container u1
- Connect to u1:3000 on container u2
- Connection successful!

On host:
```
docker network create mynetwork
```

```
docker run -it --rm --name u1 --network mynetwork ubuntu
docker run -it --rm --name u2 --network mynetwork ubuntu
```

On both container:
```
apt update
apt install -y iproute2 netcat
```

On u1:
```
root@b7bfb1fa159e:/# while true; do nc -l 3000; done
```

On u2:
```
root@1bd1cee0ce53:/# nc b7bfb1fa159e 3000 -v
Connection to b7bfb1fa159e (172.18.0.2) 3000 port [tcp/*] succeeded!
```

## Kubernetes

- Pods can reach each other, even in different namespaces.
- Containers only need to listen on a port inside, and there is not nessessary to specify the exposed port in the Dockerfile or mention the pod config.
- The `ports` field in the container object is used to list the ports to expose from the container.
- Not specifying a port here does not prevent that port from being exposed.
- Any port that is listening on the default `0.0.0.0` address inside a container will be accessible from the network.
- Modifying the ports array with a strategic merge patch may corrupt the data.
- The `ports` field is more like a self-document purpose.

Reference: https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/#ports

We can re-test it in k8s, starting the pods in different namespace.

On host:
```
kubectl create ns ns1
kubectl create ns ns2
```

```
kubectl run -n ns1 -it u1 --image=ubuntu
kubectl run -n ns2 -it u2 --image=ubuntu
```

repeat what we did in Docker - u2 is able to connect to u1:3000 with ip.
