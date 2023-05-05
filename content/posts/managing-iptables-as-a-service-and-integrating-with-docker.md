---
Title: Managing iptables as a Service and Integrating with Docker
Date: 2023-05-05

Tags:
- firewall
- iptables
---

Reference: https://upcloud.com/resources/tutorials/configure-iptables-centos

Problem: Non-persistent iptables rule after system restarts.

This is how to add a rule to iptables:
```
sudo iptables -A INPUT -p tcp --dport ssh -j ACCEPT
```

Solution:

This command applies the rule immediately, but it won't persist after a system restart.

To make iptables rules persistent, follow these steps:

1. Install iptables-services:
```
sudo yum install iptables-services
```

2. Start and enable iptables:
```
sudo systemctl start iptables
sudo systemctl enable iptables
```

3. Save the current rules to a file:
```
sudo iptables-save > /etc/sysconfig/iptables
```

To restore the rules, you can either restart iptables or use iptables-restore:
```
sudo iptables-restore < /etc/sysconfig/iptables
```

If you need to manage iptables rules alongside Docker, remember:

1. Always restart iptables before Docker:
```
sudo systemctl restart iptables
sudo systemctl restart docker
```

2. If you need to add custom rules to iptables for Docker containers, use the `DOCKER-USER` chain. This chain is designed to allow user-defined rules.

3. To add a custom rule to the `DOCKER-USER` chain, for example, allowing HTTP access to Docker containers:
```
sudo iptables -A DOCKER-USER -p tcp --dport 80 -j ACCEPT
```

4. Save and restore the rules to ensure persistence across system restarts:
```
sudo iptables-save > /etc/sysconfig/iptables
sudo systemctl restart iptables
sudo systemctl restart docker
```

By following these practices, we can manage iptables rules effectively while preventing conflicts with Docker's automatic iptables management. This approach ensures a seamless integration of iptables with Docker and maintains the security of the system.
