---
title: "AWS EKS Workshop"
date: 2021-11-27
tags:
- workshop
- aws
- eks
---

# Day1

## Setup
* https://www.eksworkshop.com/020_prerequisites/ec2instance
* https://www.eksworkshop.com/030_eksctl/launcheks
* https://www.eksworkshop.com/beginner/060_helm/helm_intro/install

## Tasks
* [Deploy simple application](https://www.eksworkshop.com/beginner/050_deploy)
* [Fargate](https://www.eksworkshop.com/beginner/180_fargate)

## Notes

There are 2 modes in EKS: 
* EC2 
* Fargate

They can coexist in EKS.

Also we can only fargate only. When we deploy plugins to the cluster, It will be deployed to the control panel. (hidden from user)

### How to use fargate in EKS:

* create a fargate profile which is associated with a namespace
* deploy yml to that namespace

### Pros and Con of fargate

#### can be scaled up quickly 

* it is a micro vm as similar as pod
* one fargate instance per pod

### lower cost of fargate

* [Fargate will be cheaper](https://aws.amazon.com/blogs/containers/theoretical-cost-optimization-by-amazon-ecs-launch-type-fargate-vs-ec2/) in some situations. 
* no need to rent large ec2 instance. it is good if you are doing batch jobs with EKS
* no need to maintain the ec2 instance (patch it, upgrade the kubelet)

### not well integrate with other AWS services

like efs?...need to further study on it.

# Day3

## Tasks
* https://www.eksworkshop.com/intermediate/245_x-ray/microservices
* https://www.eksworkshop.com/intermediate/246_monitoring_amp_amg
* https://www.eksworkshop.com/intermediate/250_cloudwatch_container_insights
* https://www.eksworkshop.com/beginner/170_statefulset
* https://www.eksworkshop.com/beginner/190_efs

## Alternative of argo
* https://www.eksworkshop.com/intermediate/260_weave_flux
* https://www.eksworkshop.com/intermediate/220_codepipeline

