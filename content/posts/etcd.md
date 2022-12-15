---
title: "Recover etcd"
date: 2021-11-26
tags:
- etcd
- k8s
---

| Control Panel |
| --- |
| svr1 |
| svr2 |
| svr3 |

## svr1

### Stop k8s

```
systemctl stop kubelet
crictl rm -f $(crictl ps -q)
```

### start the etcd in single node

```
vim /etc/kubernetes/manifests/etcd.yaml
```

```
- --force-new-cluster=true
```

### Add svr2 etcd to cluster

```
kubectl exec -it etcd-svr1.hhuge9.com -nkube-system --  sh -c 'ETCDCTL_API=3 etcdctl --endpoints 127.0.0.1:2379 --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --cacert=/etc/kubernetes/pki/etcd/ca.crt member add svr2.hhuge9.com --peer-urls="https://192.168.0.102:2380"'
```

### Save the output

```
export ETCD_NAME="svr2.hhuge9.com"
export ETCD_INITIAL_CLUSTER="svr1.hhuge9.com=http://192.168.0.101:2380,svr2.hhuge9.com=http://192.168.0.102:2380"
export ETCD_INITIAL_CLUSTER_STATE=existing
```

```
systemctl start kubelet
```

## svr2

### Stop k8s and remove etcd data

```
systemctl stop kubelet
crictl rm -f $(crictl ps -q)
rm -rf /var/lib/etcd
```

### Update etcd config (with the output in svr1)

```
vim /etc/kubernetes/manifests/etcd.yaml
```

```
- --initial-cluster=svr1.hhuge9.com=https://192.168.0.101:2380,svr2.hhuge9.com=https://192.168.0.102:2380
- --initial-cluster-state=existing
```

```
systemctl start kubelet
```
