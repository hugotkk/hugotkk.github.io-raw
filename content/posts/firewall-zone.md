---
title: "Understanding firewalld zone"
date: 2023-12-05
tags:
- firewalld
- security
- firewall
---

Topology:

- subnet100: 192.168.100.0/24
  - web01: 192.168.100.111
  - web02: 192.168.100.112
- subnet200: 192.168.200.0/24

In firewalld, there is a trusted zone:

```xml
<?xml version="1.0" encoding="utf-8"?>
<zone target="ACCEPT">
  <short>Trusted</short>
  <description>Trusted</description>
  <service name="ssh"/>
  <source address="192.168.100.0/24"/>
  <forward/>
</zone>
```

I initially expected that only `ssh` would be open to `subnet100`. However, I discovered that all ports were open to `subnet100`. 

This was due to the `target="ACCEPT"` setting, which accepts all traffic that does not match any rule in the zone. 

To rectify this, I removed `target="ACCEPT"`. Now, any traffic that does not match a rule will be rejected. 

As a result, only the `ssh` service can be accessed by `subnet100`.

Next, let's examine the following setup:

- `public`: `http`
- `subnet100`: `ssh`, `mysql`, `galera`


```xml
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>Public</description>
  <interface name="eth0"/>
  <service name="http"/>
  <forward/>
</zone>
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Trusted</short>
  <description>Trusted</description>
  <source address="192.168.100.0/24"/>
  <service name="ssh"/>
  <service name="mysql"/>
  <service name="galera"/>
</zone>
```

I anticipated that only traffic from `subnet100` would be able to access `ssh`, `mysql` and `galera`. 

`http` is accessible publicly, including `subnet100`. 

However, this was not the case - `web01` could not access `http://web02`.

This issue relates to the concept of zones in firewalld.

```bash
firewall-cmd --get-active-zone
```

```
public
  interfaces: eth0
trusted
  sources: 192.168.100.0/24
```

```bash
firewall-cmd --get-default-zone
```

```
public
```

This show there are two zones: `public` and `trusted` in firewall. 

`public` is set as the default zone. 

The key concept here is that only active zones enforce their rules. An active zone must have either an `interface` or a `source` defined in its config. 

If the traffic doesn't match any zone, it defaults to the `public` zone.

Since `web02` fall into the trusted zone, which initially did not have an `http` rule, their `http` traffic was dropped.

To resolve this, http can be added to the trusted zone config:

```xml
 <?xml version="1.0" encoding="utf-8"?>
 <zone>
   <short>Trusted</short>
   <description>Trusted</description>
   <source address="192.168.100.0/24"/>
   <service name="ssh"/>
+  <service name="http"/>
   <service name="mysql"/>
   <service name="galera"/>
 </zone>
```

To further refine the network rules, if we want to limit `http` access exclusively to `web02`, we can employ a rich rule in firewalld. 

This approach is contingent on web02 being part of the trusted zone.

```xml
 <?xml version="1.0" encoding="utf-8"?>
 <zone>
     <short>Trusted</short>
     <description>Trusted</description>
     <source address="192.168.100.0/24"/>
     <service name="ssh"/>
     <service name="mysql"/>
     <service name="galera"/>
+    <rule family="ipv4">
+      <source address="192.168.100.112"/>
+      <service name="http"/>
+      <accept/>
    </rule>
 </zone>
```

In other scenario where we want 

- `subnet200`: `http`
- `subnet100`, `subnet200`: `ssh` `mysql` and `galera`

```xml
 <?xml version="1.0" encoding="utf-8"?>
 <zone>
     <short>Trusted</short>
     <description>Trusted</description>
     <source address="192.168.100.0/24"/>
     <service name="ssh"/>
     <service name="mysql"/>
     <service name="galera"/>
+    <rule family="ipv4">
+      <source address="192.168.200.0/24"/>
+      <service name="http"/>
+      <accept/>
    </rule>
 </zone>
```

But this config will not work effectively. 

In this config, the rule specified for `subnet200` within the trusted zone will not be effective, as the zone itself does not apply to `subnet200`.

```bash
firewall-cmd --get-active-zone
```

```
public
  interfaces: eth0
trusted
  sources: 192.168.100.0/24
```

It's still 192.168.100.0/24.

To address the issue of allowing specific services for different subnets, the firewalld config can be modified by adding `subnet200` as a source in the trusted zone. 

This change allows both `subnet100` and `subnet200` to be active within the trusted zone. 

```xml
 <?xml version="1.0" encoding="utf-8"?>
 <zone>
     <short>Trusted</short>
     <description>Trusted</description>
     <source address="192.168.100.0/24"/>
+    <source address="192.168.200.0/24"/>
     <service name="http"/>
     <service name="ssh"/>
     <service name="mysql"/>
     <service name="galera"/>
     <rule family="ipv4">
       <source address="192.168.200.0/24"/>
       <service name="http"/>
       <accept/>
     </rule>
 </zone>
```

However, this config permits both subnets to access `ssh`, `mysql` and `galera`, which may not be desirable.

To control access more granularly, the config can use rich rules to specify which services are accessible by which subnet. 

Alternatively, creating a new zone can also serve this purpose. 

In this example, 
- `subnet200`: `ssh`
- `subnet100`: `mysql` `galera`
- `subnet100`, `subnet200`: `http`

```xml
 <?xml version="1.0" encoding="utf-8"?>
 <zone>
     <short>Trusted</short>
     <description>Trusted</description>
     <source address="192.168.100.0/24"/>
+    <source address="192.168.200.0/24"/>
     <service name="http"/>
+    <rule family="ipv4">
+      <source address="192.168.200.0/24"/>
+      <service name="ssh"/>
+      <accept/>
+    </rule>
+    <rule family="ipv4">
+      <source address="192.168.100.0/24"/>
+      <service name="mysql"/>
+      <accept/>
+    </rule>
+    <rule family="ipv4">
+      <source address="192.168.100.0/24"/>
+      <service name="galera"/>
+      <accept/>
+    </rule>
 </zone>
```

This config effectively segregates the services each subnet can access, ensuring a more secure and controlled network environment.
