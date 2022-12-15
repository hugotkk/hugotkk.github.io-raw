---
title: "Opsworks"
date: 2022-02-21
tags:
- cloud
- aws
- devops
- opsworks
---

## OpsWorks

* stack = cookbooks
* layer = frontend, backend, api, rds (how to config to instance)
* app = source code. can deploy and redeploy

### Stack

* I am exploring [this cookbook](https://docs.aws.amazon.com/opsworks/latest/userguide/gettingstarted-linux-explore-cookbook.html) from aws
* The [nodejs demo cookbook repo](https://github.com/aws-samples/opsworks-linux-demo-cookbook-nodejs) does not include its dependencies
* In the opsworks demo, it uses opsworks-linux-demo-cookbooks-nodejs.tar.gz as the cookbook source
* We need to convert the repo to the archive before using it
* Aftering changing the cookbook source, we need to fetch the cache by running commands on instances

Inside the opsworks-linux-demo-cookbooks-nodejs.tar.gz, we have

```
nodejs_demo
 recipes
   default.rb
```

To run this cookbook, we set `nodejs_demo::default` in the layer

### Docker Support

* eb -> multi docker / docker + elb
* opsworks -> ecs with [linux + ec2](https://docs.aws.amazon.com/opsworks/latest/userguide/workinglayers-ecscluster.html)

### Layer

* auto-healing = restart the instance when losing connection with opsworks agent
* elb
* sg
* ebs
* eip
* cloudwatch
* instance

* rds service layer
* elb service layer
* ecs cluster layer

### App

* source code
* env
* domain
* ssl
* db

source code are placed at /svr/<app_name>

### Instance Types

* 24/7 = normal instance
* time = start at specific period
* load-based = asg

Behaviors on auto-healing between these two instance type
* ebs-backed = re-create the instance
* instance = stop and start

### Issue

#### Instances are stuck in booting 

Reason: incorrect instance iam role

We can find the error message at

```
cat tail -f /var/logs/aws/opsworks/*
```

[My instance iam role is incorrect](https://github.com/hugotkk/aws-lab/commit/f99afa778655d590cc05153cc23cc8a91df4d892) because it should trust ec2.amazonaws.com instead of opsworks.

and I don't realise that opsworks need [2 iam roles](https://github.com/hugotkk/aws-lab/blob/586a36f1cf45d42d4e368279efd98901cf2f79d4/opsworks-iam.yml):
* [service role](https://docs.aws.amazon.com/opsworks/latest/userguide/opsworks-security-servicerole.html): use by opsworks. need to trust opsworks.amazonaws.com and passrole to ec2.amazonaws.com
* [instance iam role](https://docs.aws.amazon.com/opsworks/latest/userguide/opsworks-security-appsrole.html): use by ec2. need to trust ec2.amazonaws.com and passrole to opsworks.amazonaws.com
