---
title: "AWS SA PRO"
date: 2022-04-06
tags:
- cert
- exam
- aws
- sa
---

# api

* [api gateway throttling limits](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-request-throttling.html#apigateway-how-throttling-limits-are-applied) 
  1. aws throttling limit (region level) 
  1. per account 
  1. per-api per-stage (methods) 
  1. per-client (usage plan)

# application discovery service

* for migration planning
* connection type
  * [application discovery agent](https://docs.aws.amazon.com/application-discovery/latest/userguide/discovery-agent.html) -> install on server. support vm / physical server
  * [discovery connector](https://docs.aws.amazon.com/application-discovery/latest/userguide/discovery-connector.html) -> install on vCenter (is a ova)
  * [Migration Hub import](https://docs.aws.amazon.com/application-discovery/latest/userguide/discovery-import.html) => import the details directly


# asg

* will [automatically tag the instances](https://docs.aws.amazon.com/autoscaling/ec2/userguide/autoscaling-tagging.html#:~:text=The%20Auto%20Scaling%20group%20automatically%20adds%20a%20tag%20to%20instances%20with%20a%20key%20of%20aws%3Aautoscaling%3AgroupName%20and%20a%20value%20of%20the%20Auto%20Scaling%20group%20name.) by default
* [cooldown will start after last instances launched](https://docs.aws.amazon.com/autoscaling/ec2/userguide/Cooldown.html#:~:text=With%20multiple%20instances%2C%20the%20cooldown%20period%20(either%20the%20default%20cooldown%20or%20the%20scaling%2Dspecific%20cooldown)%20takes%20effect%20starting%20when%20the%20last%20instance%20finishes%20launching%20or%20terminating.) if there are multiple instance scale at the moment

# athena

* [performance tunning](https://aws.amazon.com/blogs/big-data/top-10-performance-tuning-tips-for-amazon-athena/) - 1. partition data 2. compression (glue) 3. optimise the file size (aws glue) 4. use columnar (apache orc, parquet with spark or hive on EMR) 5. prevent select * 6. use limit by ([guide for columnar](https://blog.matthewrathbone.com/2019/11/21/guide-to-columnar-file-formats.html))

# billing

* [budget](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-budgets-budget.html) - create alert if cost exceed the budget
* setup for cost analysis
  1. [enable cost allocation tags in billing](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/cost-alloc-tags.html)
  1. [allow user access the billing](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/control-access-billing.html)
* cost allocation tags => tags
  * user-define => user:XXXX
  * aws generated => aws:XXXX
* cost explorer => ui for search and filtering
* cost category => filter in cost explorer (saved filter)
* [cost budget](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/cost-alloc-tags.html) => billing alarm with foretasted charged + filtering + linked account; billing alert => amount already be charged

# cfn

* can use [automatic deployment](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-orgs-manage-auto-deployment.html) to auto deploy existing stackset to new accounts in organisation

# data pipeline

* [components](https://docs.aws.amazon.com/datapipeline/latest/DeveloperGuide/what-is-datapipeline.html)
  1. pipeline definition 
  2. pipeline schedule 
  3. task runner
* swf which is specific for data engineering
* task runner can [run on on-premise hosts](https://aws.amazon.com/datapipeline/faqs/#:~:text=How%20do%20I%20install%20a%20Task%20Runner%20on%20my%20on%2Dpremise%20hosts%3F)

# data sync

* [source location](https://docs.aws.amazon.com/datasync/latest/userguide/configure-source-location.html)
* [destination location](https://docs.aws.amazon.com/datasync/latest/userguide/create-destination-location.html)

# ddb

* support [atomic counter](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/WorkingWithItems.html)

# ebs

* aws [only recommend raid0](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/raid-config.html#:~:text=Creating%20a-,RAID%200,-array%20allows%20you)

# efs

* HA - [regional replication](https://aws.amazon.com/blogs/aws/new-replication-for-amazon-elastic-file-system-efs/) => notice that it means multi az not multi regions
* backup 
  * solution 1 - [data pipeline](https://docs.aws.amazon.com/efs/latest/ug/alternative-efs-backup.html) to backup efs in multi regions
  * solution 2 - lambda (efs region a > s3 region a > s3 region b > efs region b)
  
# elasticache

* [caching strategies](https://docs.aws.amazon.com/AmazonElastiCache/latest/mem-ug/Strategies.html#Strategies.WithTTL)
  * lazy load - set cache when select from db
  * write through - set cache when write to db
  * ttl - write through + lazy load but set an expire date
* [connection endpoints](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/Endpoints.html)
  * node ep - read and write
  * primary ep for write; reader ep for read (cluster mode disabled)
  * configuration ep for read and write like node ep (cluster mode enable)
* [automatically cache query](https://aws.amazon.com/blogs/database/automating-sql-caching-for-amazon-elasticache-and-amazon-rds/) to elasticache for rds, aurora and redshift (use proxy)

# elb

* classic load balancer only support [at most one subnet per az](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-create-internal-load-balancer.html)

# iam

* set [SAML session tags](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_abac-saml.html) for access control (add attribute to idp metadata)
* [policy to deny access on specific region](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_examples_general.html#example-scp-deny-region) - deny all except the global service
* in console, the [instance profile](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html) are automatically created along with the iam role with the same name

# mTurk

* submit a request to mTurk. outsource manual tasks like taking survey, text recognition, data migration to public

# other

* [Public Data Sets](https://aws.amazon.com/about-aws/whats-new/2008/12/03/public-data-sets-on-aws-now-available/) - data set for public access. [more details](https://aws.amazon.com/opendata/open-data-sponsorship-program/)
* [fileb://](https://aws.amazon.com/blogs/developer/best-practices-for-local-file-parameters/) is supported in 
  1. s3, kms (key)
  1. ec2 user data (gzip)
  1. s3 (encryption key)
* [govcloud comparson](https://docs.aws.amazon.com/govcloud-us/latest/UserGuide/govcloud-differences.html)
  1. billing and using can be viewed in standard account
  1. only us citizen employees can administer the govcloud
  1. authentication is isolated from amazon.com
  1. network is isolated from other region

# rds 

* [RMAN restore isn't supported for Amazon RDS for Oracle DB instances](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Appendix.Oracle.CommonDBATasks.RMAN.html#Appendix.Oracle.CommonDBATasks.BackupArchivedLogs)

# route53

* [health check](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-failover-determining-health-of-endpoints.html) must respond with 2xx or 3xx. support tcp and http
* support [DNSSEC](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/domain-configure-dnssec.html)

# s3

* access control to object c (account c) from request user a (account a) and s3 bucket (account b)
  1. check the [iam role in account a](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-auth-workflow-bucket-operation.html)
  1. check the bucket policy in account 
  1. check the [object acl in object owner](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-auth-workflow-object-operation.html)
* [event notification](https://docs.aws.amazon.com/AmazonS3/latest/userguide/NotificationHowTo.html#:~:text=Amazon%20S3%20event%20notifications%20are%20designed%20to%20be%20delivered%20at%20least%20once.%20Typically%2C%20event%20notifications%20are%20delivered%20in%20seconds%20but%20can%20sometimes%20take%20a%20minute%20or%20longer.) support object and bucket level but it will resend the notification and sometimes will delay so people use cloudwatch event instead
* there is a [check](https://docs.aws.amazon.com/awssupport/latest/user/security-checks.html#amazon-s3-bucket-permissions) in trusted advisor for the check open access in s3 bucket but no remediation for that. to fix the bucket permission automatically and use [lambda + cloudwatch event](https://aws.amazon.com/blogs/security/how-to-detect-and-automatically-remediate-unintended-permissions-in-amazon-s3-object-acls-with-cloudwatch-events/)
* cf cannot cache if the request is larger than 30gb. can use [range request](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/RangeGETs.html) to chunk the large file into smaller object
* [requester pays](https://docs.aws.amazon.com/AmazonS3/latest/userguide/RequesterPaysBuckets.html) don't support 1. anonymous request 2. SOAP request
* default 100, max 1000 in each account
* [genomics data processing use case](https://docs.aws.amazon.com/whitepapers/latest/genomics-data-transfer-analytics-and-machine-learning/transferring-genomics-data-to-the-cloud-and-establishing-data-access-patterns-using-aws-datasync-and-aws-storage-gateway-for-files.html) 
  1. sync data to s3 with data sync 
  2. use s3 for data storage 
  3. use storage gateway (on-premise access) / fsx (ec2 access)
* s3 encryption - [only support symmetric keys](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingClientSideEncryption.html#:~:text=Amazon%20S3%20only%20supports%20symmetric%20keys%20and%20not%20asymmetric%20keys.)
* when [downloading encrypted s3 object](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingClientSideEncryption.html#:~:text=that%20it%20uploads.-,When%20downloading%20an%20object,-%E2%80%94%20The%20client%20downloads), have to download the encrypted object along with a cipher blob version of the data key. client send the cipher blob to kms to get the plaintext version of data key to decrypt the object

# sso

* [permission sets](https://docs.aws.amazon.com/singlesignon/latest/userguide/permissionsetsconcept.html) - 1 permission set has multiple iam policies => associate to user / group
* sso
  * ad (identity provider) -> aws sso -> application (github, dropbox) / aws accounts
  * [ip sources](https://docs.aws.amazon.com/singlesignon/latest/userguide/connectonpremad.html) => aws sso / ad connector or aws managed ad / external ad (two way trust)
  * server -> client
  * server = adfs => create an app => config app sign-in and sign-out url
  * client = cc => trusted idp => config idP's sign-in and sign-out url + cert
  * user login with ad's app endpoint => ad post data to app's sign-in url => app receive and decrypt the data from ad and give permission to user
  * federation => single account only

# storage gateway

* storage gateway type
  * [volume](https://docs.aws.amazon.com/storagegateway/latest/userguide/StorageGatewayConcepts.html#volume-gateway-concepts) => mount as a disk (iSCSI) => s3 => ebs
    * cached => save some frequently used data to vm
    * stored => completely on s3
  * file => smb / nfs
  * tape => [tape backup software](https://aws.amazon.com/storagegateway/features/#:~:text=Tape%20Gateway%20presents%20an%20iSCSI,your%20tape%2Dbased%20backup%20workflows.)

# swf

* [swf vs step function](https://aws.amazon.com/swf/faqs/#:~:text=When%20should%20I%20use%20Amazon%20SWF%20vs.%20AWS%20Step%20Functions%3F): use step function first. if does not fit => swf
* use case: [processing large product catalog using Amazon Mechanical Turk](https://aws.amazon.com/swf/faqs/#:~:text=the%20failed%20chunks.-,Use%20case%20%232,-%3A%20Processing%20large%20product)

# vpc

* for dx, need to [enabled route propagation](https://aws.amazon.com/premiumsupport/knowledge-center/routing-dx-private-virtual-interface/#:~:text=Verify%20that%20you%27ve%20enabled%20route%20propagation%20to%20your%20subnet%20route%20tables)

# worksplace

* use [connection aliases](https://docs.aws.amazon.com/workspaces/latest/adminguide/cross-region-redirection.html) for cross-region workspaces redirection
  * create connection alias
  * share to other account
  * associate with directories in each region
  * setup route53 for failover
  * setup connection string
