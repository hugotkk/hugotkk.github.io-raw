---
title: "Three Ways to Use Kubernetes Service Account Token"
date: 2023-05-04
tags:
- k8s
- service-account
---

The token will be stored at `/var/run/secrets/kubernetes.io/serviceaccount/token` in container. ([See here](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#opt-out-of-api-credential-automounting))

Three ways to use:
- By code: Kubernetes Python library
- With kubectl binary: it will use the token automatically as your credentials. So we can do `kubectl get po` within the pod
- curl: eg:
```
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
API_ENDPOINT=$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT_HTTPS
curl --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
-H 'Authentification: Bearer $TOKEN' \
https://$API_ENDPOINT/v1/api/pods
```