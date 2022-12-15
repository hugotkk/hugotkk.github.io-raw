---
title: "EKS vs AKS"
date: 2021-11-27
tags:
- k8s
- aws
- azure
---

## Control Panel Fee

* AKS: 0
* EKS: fixed charge 0.1 USD / Hour

## Integration

I think AKS is better, most of the provisioning plugins can be enabled by one click

In aws, you need to 

* install plugins 
* setup iam service account for the plugins to access AWS service

## application routing

For AKS,
* [it](https://docs.microsoft.com/en-us/azure/aks/http-application-routing) is just an checkbox option

For EKS, you need to 

* [setup IAM OICD](https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html)
* [setup external dns](https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/aws.md)
* [setup ALB contrller](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html)

## auto scaler

Same as application routing -> [auto scaler](https://docs.microsoft.com/en-us/azure/aks/cluster-autoscaler#update-an-existing-aks-cluster-to-enable-the-cluster-autoscaler). enable and use

in AWS, we need to setup [cluster autoscaler](https://docs.aws.amazon.com/eks/latest/userguide/cluster-autoscaler.html)

## volume provisioning

AKS supports volume provisioning with Azure File (EFS) & Azure Disk (EBS), it has many pre-defined volume class...we do not need to install any plugin

For EKS, we need CSI driver

* [ebs](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html)
* [efs](https://docs.aws.amazon.com/eks/latest/userguide/efs-csi.html)

## Virtual node

It is AKS things, like EKS's fargate...but I cannot find the price..

## 2021-12-10 updates:

I have asked on Microsoft Q&A: https://docs.microsoft.com/en-us/answers/questions/659456/price-of-ake-virtual-node-and-aks-without-node-poo.html

* need min 1 system node group 
* can start / stop the cluster 
* the VM will stop charging when the cluster is stop 
* charge of other provisioned services like LB and Azure File are unknown...
* virtual node will charge as container service (like fargate)
