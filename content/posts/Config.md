---
title: "AWS Config"
date: 2022-02-21
tags:
- aws
- devops
- config
---

## config aggregator

### aggreagate all account under organization

* enable service role in organization
* set up iam with
  * viewing the organization

* service role give config.amazonaws.com access for the config resource
* additional iam right to view accounts in organization

from management account or delegated admin to use this option

### aggreagate specfic account

* authorization

## cfn stackset

add stack to stackset = deploy stack
delete stack from stackset = delete the stack

### organziation

use managed service mode
enable service role in organization
change choose account in organization

### other discount

use iam service role

administrator role: from ac
* trust cloudwatch
* can assume as execute role

executor role: to ac
* trust admintrator ac
* have access to create resource

for example
* Raphael = Cloudformation
* Wayne = Administrator (trust Cloudformation)
* Hugo = Executor (trust Adminstrator)

that will be
* Wayne trust Raphael
* Hugo trust Wayne
* Hugo can do the job
* Wayne can use Hugo

Wayne ask Raphael to ask Hugo to do the job = admin ac let cloudformation to assume hugo's right to create resource in hugo account
