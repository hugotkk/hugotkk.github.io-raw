---
title: "AWS EKS Workshop"
date: 2021-11-27
tags:
- workshop
- aws
- eks
---

# EKS Workshop

## Day1

## Setup
* https://www.eksworkshop.com/020_prerequisites/ec2instance
* https://www.eksworkshop.com/030_eksctl/launcheks
* https://www.eksworkshop.com/beginner/060_helm/helm_intro/install

## Tasks
* [Deploy simple application](https://www.eksworkshop.com/beginner/050_deploy)
* [Fargate](https://www.eksworkshop.com/beginner/180_fargate)

# Notes

There are 2 modes in EKS: 
* EC2 
* Fargate

They can coexist in EKS.

Also we can only fargate only. When we deploy plugins to the cluster, It will be deployed to the control panel. (hidden from user)

## How to use fargate in EKS:

* create a fargate profile which is associated with a namespace
* deploy yml to that namespace

## Cost of fargate

[Fargate will be cheaper](https://aws.amazon.com/blogs/containers/theoretical-cost-optimization-by-amazon-ecs-launch-type-fargate-vs-ec2/) in some situations. 

## Pros and Con of fargate

It is a micro vm so it can be scaled up quickly. (I think it is a pod)

one fargate instance per pod

But I remember that it does not well support integration with other AWS services, like efs...need to further study on it.
