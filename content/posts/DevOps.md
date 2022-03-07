---
title: "AWS Certified DevOps Engineer - Professional"
date: 2022-03-06
tags:
- aws
- devops
---

# DevOps choices

## Deployment
* faster boot time - opsworks slower; ami faster
* using chef - opsworks
* need to update config when new node online - opsworks [Configure lifecycle event](https://docs.aws.amazon.com/opsworks/latest/userguide/workingcookbook-events.html)
* less administrative overhead: eb > cloudformation when both solutions works
* auto healing: [opsworks](https://docs.aws.amazon.com/opsworks/latest/userguide/workinginstances-autohealing.html), codedeploy, eb (bcoz of the asg)
* rolling: eb, opsworks (not ideal, it is [manual deploy](https://docs.aws.amazon.com/opsworks/latest/userguide/best-deploy.html#best-deploy-rolling)), cloudformation+asg+[AutoScalingRollingUpdate policy](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-attribute-updatepolicy.html#cfn-attributes-updatepolicy-rollingupdate), codedeploy
* rolling = drop traffic to n instance > deploy > allow traffic
* in-place = deploy the deploy to all instances (parallel)
* blue/green deployment: eb ([cname swap](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.CNAMESwap.html)), codedeploy, 2x(cfn+asg+elb)+route53 or 2(cfn+asg)+elb(weighted target groups)
* blue/green deployment and want to [delay the old asg termination](https://docs.aws.amazon.com/codedeploy/latest/APIReference/API_BlueInstanceTerminationOption.html): codedeploy
* canary deployment: codedeploy only on [lambda](https://docs.aws.amazon.com/codedeploy/latest/userguide/deployment-configurations.html#deployment-configuration-cloudformation-bg) / [ecs](https://docs.aws.amazon.com/codedeploy/latest/userguide/deployment-configurations.html#deployment-configuration-ecs), eb ([traffic splitting](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.rolling-version-deploy.html#environments-cfg-trafficsplitting-method)), api gateway
* eb's [immutable deployment](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environmentmgmt-updates-immutable.html): create 2nd asg. deploy code to new asg and create new resources in batch > delete old asg after the deployment. (kind of rolling deployment. not a blue/green, the new resource will accept the traffic during the deployment)
* a/b test for a long time: (cloudformation+asg+alb)x2+route53 weighted round robin
* multi app, multi dependencies: use docker: cfn, eb
* rollback by CloudWatch alarm: codedeploy, eb not work (only by health check)
* some nodes are not updated after a successful codedeploy deployment: asg create new node during deployment
* codedeploy sucks in the lifecycle event hook (ec2 ~1hr): [script error](https://docs.aws.amazon.com/codedeploy/latest/userguide/troubleshooting-deployments.html#troubleshooting-deployments-lifecycle-event-failures)
* codedeploy sucks at the AllowTraffic lifecycle event: [elb health check failed](https://docs.aws.amazon.com/codedeploy/latest/userguide/troubleshooting-deployments.html#troubleshooting-deployments-lifecycle-event-failures)
* opsworks sucks at booting: [agent doesn't start or incorrect iam role in instance profile](https://docs.aws.amazon.com/opsworks/latest/userguide/common-issues-troubleshoot.html#common-issues-troubleshoot-booting)
* all lifecycle event skipped in codedeploy: [agent doesn't start / security group blocked the communication](https://docs.aws.amazon.com/codedeploy/latest/userguide/troubleshooting-deployments.html#troubleshooting-skipped-lifecycle-events)
* limit the resources on cfn launching: use catalog (like a marketplace). more iam control to the cloudformation template. With cloudfromation, you cannot limit user upload what cloudformation template.

## Backup & Restore

* cross region efs backup: lambda: (at region1) use ec2 with efs mount put data to region2 s3 -> (region2) ec2 with efs mount pull data from s3

## Cloudformation

* work as teams / multi tiers (network, security, application): [nested stack](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-stack.html#aws-properties-stack--examples) / [importvalue](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-importvalue.html)

## ASG

* troubleshoot asg instance: [standby](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-suspend-resume-processes.html) / [asg lifecycle hook](https://aws.amazon.com/premiumsupport/knowledge-center/auto-scaling-delay-termination/) (1hr only)
* handle predictable traffic: [scheduled scaling](https://docs.aws.amazon.com/autoscaling/ec2/userguide/schedule_time.html)
* prevent scale-in: [instance scale-in protection](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-instance-protection.html)
* sending log / licence register deregister: [asg lifecycle hook](https://docs.aws.amazon.com/autoscaling/ec2/userguide/lifecycle-hooks.html)

## Data analysis / Loggings

* batch jobs / reporting (like spark): EMR
* visualise data (like BI): Redshift / Quicksight (cost effective)
* log searching: elastic search (ES) (renamed to opensearch)
* query S3 data: athena
* report time slow: offload the job to other application(like lambda) with kinesis stream / scale-up the cluster with asg
* apache hive ~= aws glue

## DB

* dynamodb stream = kinesis stream (more advance)
* throttle on dynamodb stream: limited to 2 connections at the moment. use 1 lambda > sns > other lambda(s)
* dynamodb with many read: [dynamodb accelerator](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DAX.html) (like redis)
* multi-region read & write: dynamodb global table
* multi-region read only: aurora
* short DR time: read replica -> promote (/[aurora global database](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database-connecting.html))
* long DR time (few hours): lambda to backup and restore
* [conditionalcheckfailedexception](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Programming.Errors.html#:~:text=ConditionalCheckFailedException) at dynamodb: too many write on same record
* data inconsistency in dynamodb: need to use [strong consistent read](add)
* DB security: [auth with iam](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.html)

## Config

* [config aggreation](https://docs.aws.amazon.com/config/latest/developerguide/aggregate-data.html): use stackset to enable config cross accounts > assign a dedicated administrator > auth config aggregator (like peer connection, request at one side and accept at other account) | use org organisation
* [config organisation rule](https://docs.aws.amazon.com/config/latest/developerguide/config-rule-multi-account-deployment.html): can use [this](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/configservice/put-organization-config-rule.html) to put rules to all account in organisation

# Application Discovery Service

## Setting up

* [Discovery Connector](https://docs.aws.amazon.com/application-discovery/latest/userguide/discovery-connector.html) - agentless - install on vmware centre
* [Discovery Agent](https://docs.aws.amazon.com/application-discovery/latest/userguide/discovery-agent.html) - agent - install on host

# CodeCommit

## approval rule (pull request)

* targets: branch
* approval pool members (iam user)

## notification

* targets: sns / aws chatbot
* event: any activities (push, merge, delete branch...)

## trigger

* targets: sns / lambda
* event: push branches or tags

# api gateway

* can do canary deploy

## targets

* lambda
* step functions
* http
* event
* sqs
* kinesis data stream

# config

* trigger type: configuration changes / periodic
* scope: aws resource > ec2:securitygroup

## notifications

* Settings > Delivery method > sns topic - give you all changes (summary)
* Settings > Amazon Cloudwatch Events rule - good for watching specific resource config change

# Events

* can send to another account
* can send to another account in organisation

# Cloudwatch

## create alarm from logs

* logs > metric filter > metric > alarm > sns

## send logs to other place for analysis

* logs > subscription filter > lambda (cannot cross account, kinesis can) > s3 > athena
* logs > subscription filter > kinesis firehose > s3 > athena
* logs > subscription filter > kinesis stream > kinesis firehose > s3 > athena

# Kinesis

* kinesis stream -> real time data stream (like enhanced version of DYNAMODB Stream) for analysis / aggregation
* kinesis stream firehose -> for storage (s3) but can do some pre-processing

## Supported Writer & Reader

* [aws sdk / agent / library (KPL)](https://docs.aws.amazon.com/streams/latest/dev/building-producers.html) > stream > [library (client) / firehose / lambda](https://docs.aws.amazon.com/streams/latest/dev/building-consumers.html)
* [aws sdk / agent / stream / CloudWatch event / CloudWatch logs](https://docs.aws.amazon.com/firehose/latest/dev/basic-write.html) > firehose > [S3 / ES / Redshift / MongoDB / Splunk](https://docs.aws.amazon.com/firehose/latest/dev/create-destination.html) 

# Security

* GuardDuty: threat detection
* Macie: data level eg: s3
* Security Hub: give advises / integrated with different aws products like 
* inspector: cvs scanning / hardending (cis)

# S3 

## Cross account replication

* AcctA BuckA
* AcctB BuckB
* iam role in acctA
  * trust s3 to assume role
  * give permission to s3 to get buckA object
  * give permission to acctB to replica buckA object
  * give permission to s3 to encrypt & decrypt buckA object
* bucket policy in acctB
  * allow roleA to replica and put object to buckB

# Cloudformation

## Custom Resource

* [bind to lambda / sns](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-cloudformation-customresource.html#aws-resource-cloudformation-customresource--examples)
* need to handle the [create / update / delete event from the stack](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/crpg-ref.html)

# ECS

## AMI

use [ECS-optimised AMI](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-optimized_AMI.html) - have container agent installed

## loggings

* container log - [awslogs](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/using_awslogs.html) - need container agent
* system log - [cloudwatch agent](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/using_cloudwatch_logs.html)
