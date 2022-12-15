---
title: "Istio Note"
date: 2022-09-06
tags:
- k8s
- service-mesh
- istio
---

## Main concept

Injecting a proxy in front of each pods - so we can do traffic management 
* Traffic Shifting
* Fault Injection
* Circuit Breaking
* Mirror

and apply security features
* mtls between pods
* security policy

or some logging / auditing works

Imagine there is a proxy server in front of each pod. 

VirtualService and DestinationRule are the configure of the proxy server

Actually the offical documentation is good enough. here is a good
[course](https://www.udemy.com/course/istio-hands-on-for-kubernetes/) to learn about istio.


## Istio Settings

* [min tls protocol](https://istio.io/latest/docs/tasks/security/tls-configuration/workload-min-tls-version/)
* [memory and cpu limit on istiod](https://istio.io/latest/docs/setup/additional-setup/customize-installation/#customize-kubernetes-settings) - I use this to reduce the memory limit on istiod. The default profile is 2G memory by default.
* [block un-registry egress by default](https://istio.io/latest/docs/tasks/traffic-management/egress/egress-control/#change-to-the-blocking-by-default-policy) - See Service Entry
* [dns proxing](https://istio.io/latest/docs/ops/configuration/traffic-management/dns-proxy/) - See Service Entry
* [enable egress gateway](https://istio.io/latest/docs/tasks/traffic-management/egress/egress-gateway/#deploy-istio-egress-gateway)

## Install Multicluster

[Setup Guide](https://istio.io/latest/docs/setup/install/multicluster/)

The `Before you Begin` section is important. For Multi-Primary Multicluster, we need to generate a root certificate to issue the certificates for cluster1 and cluster2. See [Configure Trust](https://istio.io/latest/docs/setup/install/multicluster/before-you-begin)

* Multi-primary: istiod on each cluster
* Primary-remote: only primary will have istiod - remote will share the istiod with primary
* On different networks: the pods are in different networks at 2 k8s clusters. EKS support [vpc networking](https://docs.aws.amazon.com/eks/latest/userguide/pod-networking.html), the pods can be in the vpc subnet so the pods in different cluster can be configured on the same network. I have set up 2 k3s clusters on aws ec2. The pods are on different networks.


## Install VM Workload

istio can bring VM into the service mesh. treat vm as a k8s pod

* Workload Entry: Pod
* Workload Group: Deployment
* Service Entry: Service

I follow this [guide](https://istio.io/latest/docs/setup/install/virtual-machine/) to set this up

There are few options for the installation. I chose

- Multi Network - as the vm and k8s pods are in different networks (For example: vm 192.168.0.1/24 k8s pods 10.0.1.0/24. In AWS, we pods can use the aws vpc network which mean if we have a subnet 192.168.0.1/24 both vm and eks pods can be in same subnet)
- Automated WorkloadEntry Creation - `workload entry` will automatically create after joining the mesh

### dns

Before the installation, we need to setup a dns server first. VM need to know about the k8s cluster (\*.cluster.local eg: httpbin.default.svc.cluster.local).

Here are some notes on setting up dnsmasq on ec2.

/etc/dnsmasq.conf

```
bind-interfaces
listen-address=127.0.0.1
server=169.254.169.254
address=/cluster.local/<k8s_cluster_ip>/
```

* bind-interfaces + listen-address make the dns server work. bind-interfaces will bind port 53 on all interfaces but listen-address will restrict it on specific interface
* need to forward queries to aws's dns (169.254.169.254)

/etc/resolv.conf

```
nameserver 127.0.0.1
```

update resolv.conf to tell the vm to use the dnsmasq

For ubuntu, the resolv.conf will be managed by systemd-resolved so it is better to disable it

```
systemctl disable systemd-resolved
systemctl stop systemd-resolved
```

system-resolved will create a symbolic link to /etc/resolv.conf. When systemd-resolved is disable, the resolv.conf may disappear after the server restart. So be careful of that.

### Install with revision

If istio is installed with the canary method, the istiod service name will be different.

* without revision: istiod.istio-system.svc.cluster.local
* with revision: istiod-\<revision>.istio-system.svc.cluster.local

The templates at `Expose services inside the cluster via the east-west gateway` are using istiod.istio-system.svc so we have to change it manually

Also we have to add `--revision` when deploy the east west gateway

### Service vs ServiceEntry

* the vm can be selected with Service - ServiceEntry is not necessary
* If we want to use ServiceEntry, [DNS proxy](https://istio.io/latest/docs/ops/configuration/traffic-management/dns-proxy/) need to be enabled

## Upgrade

### In-placed

run `istioctl upgrade` with the updated `istioctl` binary

### Canary

For example, we want to upgrade istio from 1.13.0 to 1.14.3. 

We can
* install new version (let 2 versions coexist)
* switch to new version
* uninstall the old version

Firstly, istio has to be installed with `--revision`

```
istioctl operator init --revision 1-13-0
```

and apply `IstioOperator` config with specific revision

```
spec:
...
  revision: 1-13-0
```

To enable auto injection in namespace (example is `default`), instead of

```
kubectl label ns default istio-injection=enabled
```

we have to specify the revision

```
kubectl label ns default istio.io/rev=1-13-0
```

To upgrade istio:
* Download 1.14.3 istioctl binary
* init 1.14.3 operator `istioctl operator init --revision 1-14-3`
* apply IstiOperator config with `revision: 1-14-3`
* Update injection label - `kubectl label ns default istio.io/rev=1-14-3 --overwrite`
* Uninstall the old version - `istioctl x uninstall --revision 1-13-0`


## Sidecar injection

* cannot enable globally - not like [mTLS](https://istio.io/latest/docs/tasks/security/authentication/authn-policy/#globally-enabling-istio-mutual-tls-in-strict-mode) - need to be apply [per namespace / per pod](https://istio.io/latest/docs/setup/additional-setup/sidecar-injection/#controlling-the-injection-policy)
* `hostNetwork: true` [in pod](https://istio.io/latest/docs/ops/common-problems/injection/) will stop the injection
* injection can be override in pod level

## Service Entry

This is confusing me so much.

### Why the hostname resolution does not work?

My expectation: like k8s' service but it's for external services - so we are adding an A record to the k8s DNS

For example:

```
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: mysql
spec:
  hosts:
  - db.hhuge9.com
  location: MESH_EXTERNAL
  endpoints:
  - XXXXX
```

Expected that I can connect to my own mysql service with `mysql -h db.hhuge9.com` at any pod in the mesh after apply this config

But the result is - `db.hhuge9.com` cannot be resolved

The reason is - `ServiceEntry` will not do anything on the dns unless we enable the [DNS proxying](https://istio.io/latest/docs/ops/configuration/traffic-management/dns-proxy/) feature

When we apply this `ServiceEntry`

```
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: google
spec:
  hosts:
  - google.hhuge9.com
  location: MESH_EXTERNAL
  resolution: DNS
  endpoints:
  - 142.250.200.14
```

with enabling dns proxying:

* `ISTIO_META_DNS_CAPTURE: true` - will add entry to k8s dns. `google.hhuge9.com` will be resolved with the endpoints ips (`142.250.200.14`)
* `ISTIO_META_DNS_AUTO_ALLOCATE: true` - will allocate an internal ip (like service in k8s). `google.hhuge9.com` will be resolve with that ips

We can leave `ISTIO_META_DNS_AUTO_ALLOCATE: false` and assign an internal ip at the addresses field

```
...
  hosts:
  - google.hhuge9.com
  addresses:
  - 192.168.0.1
  ...
  endpoints:
  - 142.250.200.14
```

Therefore nslookup `google.hhuge9.com` will return `192.168.0.1`

But for the host name resolution, we need `ISTIO_META_DNS_CAPTURE: true`

### Is it just a canonical name / A record?

I doubted why `ServiceEntry` is needed. We can access the service directly without applying a config, right? It looks so useless

Acutualy, It becomes useful when we want to [control the egress traffic ](https://istio.io/latest/docs/tasks/traffic-management/egress/egress-control/#change-to-the-blocking-by-default-policy)

When the `REGISTRY_ONLY` policy is enabled, only registered services can be accessed within the mesh. Therefore, a service entry of `facebook.com` will be needed if we want to access facebook inside the pod


## AuthorizationPolicy

* Useful examples can be found on [Concept > Security > Authorization policies](https://istio.io/latest/docs/concepts/security/#authorization-policies) and [Reference > Security > AuthorizationPolicy](https://istio.io/latest/docs/reference/config/security/authorization-policy/)
* Best practise is applying `allow-nothing` first then apply our policies
* `allow-all` and `deny-all` are useful in debug

```
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
 name: httpbin
 namespace: foo
spec:
 selector:
   matchLabels:
     app: httpbin
     version: v1
 action: ALLOW
 rules:
 - from:
   - source:
       principals: ["cluster.local/ns/default/sa/sleep"]
   - source:
       namespaces: ["dev"]
   to:
   - operation:
       methods: ["GET"]
   when:
   - key: request.header[foo]
     values: ["bar"]
```

This policy means - allow access to the pod in namespace `foo` with label `httpbin` and `v1` with the existence of
- GET request and
- foo: bar header and the source is
- from namespace `dev` or
- with principals `cluster.local/ns/default/sa/sleep`

`principal: cluster.local/ns/default/sa/sleep` means `sleep.default.svc.cluster.local`. It is the service account name when the mtls is enabled

The account name can be confirmed by printing out the environment variables with `env` in the pod

## Ingress Gateway

* [http](https://istio.io/latest/docs/tasks/traffic-management/ingress/ingress-control/)
* [https - tls termination on proxy](https://istio.io/latest/docs/tasks/traffic-management/ingress/secure-ingress/)
* [https - tls termination on pod](https://istio.io/latest/docs/tasks/traffic-management/ingress/ingress-sni-passthrough/) (tls passthrough)

## Egress Gateway

* [http](https://istio.io/latest/docs/tasks/traffic-management/egress/egress-gateway/#egress-gateway-for-http-traffic)
* [https](https://istio.io/latest/docs/tasks/traffic-management/egress/egress-gateway/#egress-gateway-for-https-traffic)
* [initial the request with http but connect to the target with https](https://istio.io/latest/docs/tasks/traffic-management/egress/egress-gateway-tls-origination/)

## Traffic Management

Most of the examples can be found on [here](https://istio.io/latest/docs/tasks/traffic-management/)

* Fault Injection - like: return 500 / add delay to the requests
* Traffic Shifting - eg: 50% to v1; 50% to v2 - good for canary / blue green deployment / AB testing
* Circuit Breaking - exclude the pods if face consecutive 5XX error / set max connections
* Mirror - mirror the traffic. good for logging / testing / monitoring
