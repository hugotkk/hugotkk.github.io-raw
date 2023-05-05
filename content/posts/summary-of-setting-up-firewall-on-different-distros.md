---
Title: "Summary of Setting up Firewall on Different Distros"
Date: 2023-05-05
Tags:
- iptables
- firewalld
- ufw
- firewall
---

In this article, I will cover the basics of setting up firewalls on different distros, including firewalld, ufw, and iptables. 

We will also discuss the daily tasks of firewall management, such as allowing or blocking specific source addresses/destination ports, port forwarding, and NAT.

## Web Application Example:

- A web server at `192.168.0.100`
- A node.js app at `192.168.0.200:8000` that is not publicly accessible

Tasks:

### Allow port 80 from public:

```
iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
```

```
firewall-cmd --zone=public --add-port=80/tcp --permanent
```

```
ufw allow 80/tcp
```

### Allow SSH only from a trusted IP, e.g., 43.164.66.12:

```
iptables -A INPUT -p tcp -s 43.164.66.12 --dport 22 -j ACCEPT
```

```
firewall-cmd --zone=public --add-rich-rule 'rule family="ipv4" source address=43.164.66.12 port port=22 accept'
```

```
ufw allow from 43.164.66.12 to any port 22/tcp
```

### Forward port `8080` requests to `192.168.0.200:8000`:

```
iptables -t nat -A PREROUTING -p tcp --dport 8080 \
       -j DNAT --to-destination 192.168.0.200:8000
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

```
firewall-cmd --zone=public --add-masquerade
firewall-cmd --zone="public" --add-forward-port=port=8080:proto=tcp:toaddr=198.51.100.0:toport=8000
```

### Drop traffic from bad IP, e.g., 233.228.5.86:

```
firewall-cmd --zone=public --add-rich-rule 'rule family="ipv4" source address=233.228.5.86 drop'
```

```
ufw deny from 233.228.5.86
```

When working with the NAT table, we need to enable ip_forward as well in /etc/ufw/sysctl.conf:

```
net.ipv4.ip_forward = 1
```

## NAT Example:

- A Linux router with IP `192.160.0.100` with internet access
- Internal servers `192.160.0.200`, `192.160.0.201`, `192.160.0.203` using `192.160.0.100` as router.

Tasks:

### Config NAT for internal server to have internet access:

```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

```
firewall-cmd --zone=public --add-masquerade
```

By following these guidelines, you can set up your firewall effectively and ensure the security of your system.

## Reference:

- iptables
  - [allow/deny with filter](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html-single/security_guide/index#sect-Security_Guide-Securing_Portmap-Protect_portmap_With_iptables)
  - [NAT & port forwarding](https://www.karlrupp.net/en/computer/nat_tutorial)
- [firewalld](https://www.linode.com/docs/guides/introduction-to-firewalld-on-centos/)
- [ufw](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-18-04)
