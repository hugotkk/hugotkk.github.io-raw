---
title: "k8s Authentication"
date: 2021-11-26
tags:
- k8s
- auth
---

# with IAM

user -> with API_KEY & API_TOKEN
audience attached iam role (eg: ec2 instance)

update the iam role and user mapping in configmap aws-auth

our side have aws access -> map to k8s's system to authenticate

# OIDC

OIDC built in at the eks cluster

Then OICD can request AWS IAM to issue the web-identity and use that web-identity to assume role and access AWS service

| Item |  |
| --- | --- |
| OICD | oidc.hhuge9.com |
| IAM user | hugotse |
| OIDC user | kevintse |
| IAM role | S3Admin |

## To assume a role in EKS

* [Associate OIDC to AWS IAM](https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html) - that why it is called associate-iam-oidc-provider.
* [Create an role which has a trust relationshop to the service account from OIDC to assume with WebIdentity](https://docs.aws.amazon.com/eks/latest/userguide/cluster-autoscaler.html#ca-create-policy) - so  who in OIDC can use webidentity to access aws service

## Difference between assume-role and assume-role-with-web-identity
* `hugotse` can use `aws sts assume-role` to assume `S3Admin`
* `kevintse` can use `aws sts assume-role-with-web-identity` with `web-identity` to assume `S3Admin`

## How a pod to access aws service

EKS has some webhooks, it will monitor our changes on cluster. When it see the service account associate to a pod and that account annotate with iam arn, it will

* webhook - send OICD token to AWS IAM <- How to obtain OICD token?
* AWS IAM - verify the OICD token and return webidentity to webhook
* webhook save the webidentity and mount it as a volume in the pod
* Application - use webidentity with `aws sts assume-role-with-web-identity` to access AWS Service

My question is how OICD know the serviceaccount's identity. I guess all account in EKS has a OICD token assocated

 We can assume role by a iam user
* user hugotse have right to use assume-role (call aws sts assume-role)
* role has a trusted relationship to user hugotse to assume role

user need access key / role to assume role

Or assume role by a user in oidc associated to iam
* role trusted user in oidc to assume with webidentity

oidc user need webidentity to assume role

user in oidc get webidentity from aws iam
user in oidc use the webidentity to assume role
