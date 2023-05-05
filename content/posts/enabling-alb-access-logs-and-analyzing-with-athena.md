---
title: "Enabling ALB Access Logs and Analyzing with Athena"
date: 2023-05-05
tags:
- aws
- athena
- alb
- cloud
---

Enable ALB access logging for debugging:
- Follow this guide: https://docs.aws.amazon.com/elasticloadbalancing/latest/application/enable-access-logging.html

Access logs storage:
- Logs are saved in S3
- Difficult to view directly

Log format:
- Not written to a single file
- New file with timestamp after a set interval

Solution: Set up Athena to query logs
- Follow this guide: https://docs.aws.amazon.com/athena/latest/ug/application-load-balancer-logs.html
