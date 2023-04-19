---
title: "SSH Cerificate"
date: 2023-04-06
tags:
- ssh
---

* Create a Certificate Authority (CA)
* Issue certificates for authorized users
* Define CA public key and principals name in `~/.ssh/authorized_keys` instead of users' public keys
* Use key (`~/.ssh/id_rsa`) with CA-issued certificate (`~/.ssh/id_rsa-cert.pub`) to SSH into server
* Per-account SSH certificate setup under `~/`; can also be configured at system level (`/etc/ssh/sshd_config`)

* Advantages:
  * Key rotation simplified; admin issues new certificate for new key, no need to update public keys on server

You can follow the full procedure for using a CA with SSH at [https://www.lorier.net/docs/ssh-ca.html](https://www.lorier.net/docs/ssh-ca.html) or [Creating SSH CA Certificate Signing Keys](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/sec-creating_ssh_ca_certificate_signing-keys).