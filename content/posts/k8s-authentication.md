---
title: "k8s Authentication"
date: 2021-11-26
tags:
- k8s
- auth
---

## with IAM

* user -> with API_KEY & API_TOKEN
* audience attached iam role (eg: ec2 instance)
* update the iam role and user mapping in configmap aws-auth

## OIDC

OIDC built in at the eks cluster

* OICD can request AWS IAM to issue the web identity token
* the token will be used to assume role and access AWS service

| Item | Value |
| --- | --- |
| OICD | oidc.hhuge9.com |
| IAM user | hugotse |
| OIDC user | kevintse |
| IAM role | S3Admin |

### To assume a role in EKS

* [Associate OIDC to AWS IAM](https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html) - that why it is called associate-iam-oidc-provider.
* [Create an role which has a trust relationshop to the service account from OIDC to assume with Web Identity Token](https://docs.aws.amazon.com/eks/latest/userguide/cluster-autoscaler.html#ca-create-policy) - so  who in OIDC can use web identity token to access aws service

### Difference between assume-role and assume-role-with-web-identity

* `hugotse` can use `aws sts assume-role` to assume `S3Admin`
* `kevintse` can use `aws sts assume-role-with-web-identity` with `web-identity` to assume `S3Admin`

### How a pod to access aws service

EKS has some [webhooks](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/), it will monitor our changes on cluster

You can create a custom admission webhooks, deny / modify specfic requests of api servers

When the admission webhook detects a service account associated to a pod and that account annotate with iam arn, it will

* admission webhook -> send **OICD token** to AWS IAM 
* AWS IAM -> verify the **OICD token** and return web identity token to the webhook
* admission webhook save the web identity token and mount it as a volume in the pod
* Application -> use the web identity token with `aws sts assume-role-with-web-identity` to access AWS Service

### Where does the OICD token come from?

with [Service Account Token Volume Projection](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#service-account-token-volume-projection), any pod attached with a service account will automatically mount with a OICD token

### Ways to assume role 

* if you are an iam user, use access key
* if you are in aws services (like ec2 instance), attach iam role
* or with oidc, use web identity token
  * oidc user get web identity token from aws iam with oidc token
  * oidc user use the token to assume role

### Permission needs to assume role

#### by a iam user

* user `hugotse` have right to use assume-role (call aws sts assume-role)
* a role has a trusted relationship to user `hugotse` to assume role

#### by a oicd user

* oidc user `kevintse` have right to use assume-role-with-web-identity
* a role has a trusted relationship with oicd user `kevintse` to assume role with web identity
