---
title: "Setting Up Google Auth on SSH on CentOS-7"
date: 2023-05-04
tags:
- ssh
- google-authentication
- 2fa
- security
---

Follow Digital Ocean tutorial for the setup: https://www.digitalocean.com/community/tutorials/how-to-set-up-multi-factor-authentication-for-ssh-on-centos-7

Notes on `/etc/ssh/sshd_config`:
- With `UsePAM yes`, both `password` and `keyboard-interactive` follow `/etc/pam.d/sshd`
- `PasswordAuthentication yes` is the same as `AuthenticationMethods password`

Differences between AuthenticationMethods `password` and `keyboard-interactive`:
- `password` accepts only 1 response from the user
- `keyboard-interactive` accepts multiple responses from user

Enforcing both `password` and `Google Authenticator` to login:
- Use `keyboard-interactive` to ensure successful authentication with 2FA and password

Prompt differences:
- password: `(hugo@192.168.0.14) Password:`
- keyboard-interactive: `Password:`

Time synchronization:
- Crucial for accurate authentication
- For Docker, ensure the host's clock is synchronized
