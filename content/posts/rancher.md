---
title: "Review of Rancher"
date: 2021-12-13
tags:
- k8s
- rancher
---

https://www.youtube.com/watch?v=LK6KbAlQRIg

* provide a GUI interface manage k8s clusters

# Cons (21:12)

* Still using docker (should use cri-o)
* Only support old version k8s (1.18)
* Mutable approach (not using image to provision, it installs docker and setup the cluster for you)
* Very slow to create the cluster 

# Pros
* Manage multiple clusters
* GUI
