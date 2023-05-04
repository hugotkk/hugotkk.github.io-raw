---
title: "k8s Volumes: HostPath and EmptyDir Explained"
date: 2023-05-04
tags:
- k8s
- volume
- mount
---


1. HostPath:
- Shares storage between nodes
- Can be used for data persistence across container restarts

2. EmptyDir:
- Independent for each pod
- Shared between containers within a pod
- Data is not persisted across container restarts
