---
title: "AWS SA Professional"
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
* three type of endpoint
  1. edge-optimized (default) - route to nearest cloudfront
  1. regional
  1. private

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

* [performance tunning](https://aws.amazon.com/blogs/big-data/top-10-performance-tuning-tips-for-amazon-athena/) 
  1. partition data
  2. compression (glue)
  3. optimise the file size (aws glue)
  4. use columnar (apache orc, parquet with spark or hive on EMR)
  5. prevent select * 
  6. use limit by ([guide for columnar](https://blog.matthewrathbone.com/2019/11/21/guide-to-columnar-file-formats.html))

# billing

* cost allocation tags - [tags](https://www.youtube.com/watch?v=AmvMEP_eUck) will show in the cost & usage report
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
* [billing alert](https://aws.amazon.com/blogs/aws/monitor-estimated-costs-using-amazon-cloudwatch-billing-metrics-and-alarms/#:~:text=includes%20usage%20charges%20for%20things%20like%20Amazon%20EC2%20instance%2Dhours%20and%20recurring%20fees%20for%20things%20like%20AWS%20Premium%20Support) include 
  * recurring fee like premium support 
  * ec2 instance-hours but exclude
  * one off fee 
  * refund
  * forecast

# cf

* access control
  * cf + waf + elb, it should be [cf (set custom header) > waf (validate the rule) > alb](https://aws.amazon.com/blogs/security/how-to-enhance-amazon-cloudfront-origin-security-with-aws-waf-and-aws-secrets-manager/#:~:text=In%20this%20blog%20post%2C%20you,it%20sends%20to%20your%20origin)
  * cf + s3, => [cf with oai > s3 bucket policy](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html)
  * cf + alb => [cf with custom header > alb rule](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/restrict-access-to-load-balancer.html)
* reason of cfn with s3 [access denied errors](https://aws.amazon.com/premiumsupport/knowledge-center/s3-rest-api-cloudfront-error-403/#:~:text=If%20your%20distribution%20is%20using%20a%20REST%20API%20endpoint%2C%20verify%20that%20your%20configurations%20meet%20the%20following%20requirements%20to%20avoid%20Access%20Denied%20errors%3A)
  * s3 block public access must turn off if no oas policy is set - because it will override the permissions that allow public read access
  * if request pays is turn on, the request must include the payer header
  * object cannot be kms encrypted

# cfn

* can use [automatic deployment](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-orgs-manage-auto-deployment.html) to auto deploy existing stackset to new accounts in organisation

# cloudhsm

* need [tcp/3389 for windows and tcp/22 for linux](https://docs.aws.amazon.com/cloudhsm/latest/userguide/configure-sg-client-instance.html) to connect to ec2 to install cloudhsm client; tcp/2223-2225 to communicate with the cluster

# cloudtrail

* [best practice](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/creating-trail-organization.html#creating-an-organizational-trail-best-practice) to migrate to org trail
  1. [create org trail in central account](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-create-and-update-an-organizational-trail-by-using-the-aws-cli.html)
      * create bucket for org (need to set bucket policy to allow member account to write to it)
      * enable cloudtrail feature in org
      * create org trail through cli
  1. move old trail data from member accounts to org trail bucket
  1. stop cloudtrail in member accounts and remove the old trail buckets

# codecommit

* data protection 
  * use [macie](https://docs.aws.amazon.com/codecommit/latest/userguide/data-protection.html) => can help protecting data in s3

# codedeploy

* need to connect to s3 and codedeploy endpoints

# cw

* [cw embedded metric format](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Embedded_Metric_Format_Specification.html) => can automatrically create metric from log
* cw endpoints => monitoring.us-east-2.amazonaws.com
  * monitoring.XXXX
  * no az
* treats each unique combination of dimensions as a [separate metric](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch_concepts.html#:~:text=CloudWatch%20treats%20each%20unique%20combination%20of%20dimensions%20as%20a%20separate%20metric%2C%20even%20if%20the%20metrics%20have%20the%20same%20metric%20name), even if the metrics have the same metric name

# data pipeline

* [components](https://docs.aws.amazon.com/datapipeline/latest/DeveloperGuide/what-is-datapipeline.html)
  1. pipeline definition 
  2. pipeline schedule 
  3. task runner
* swf which is specific for data engineering
* task runner can [run on on-premise hosts](https://aws.amazon.com/datapipeline/faqs/#:~:text=How%20do%20I%20install%20a%20Task%20Runner%20on%20my%20on%2Dpremise%20hosts%3F)
* task runners can be run on [any compute resources](https://docs.aws.amazon.com/datapipeline/latest/DeveloperGuide/dp-how-remote-taskrunner-client.html) (ec2 and on-premise servers)
* [use resources in multiple regions](https://docs.aws.amazon.com/datapipeline/latest/DeveloperGuide/dp-manage-region.html)
* only supported in limited region

# data sync

* use for transfer data between on-premise and aws or between aws service
* support cross-region (s3 <=> s3, efx <=> efx, efs <=> efs) sync
* [source location](https://docs.aws.amazon.com/datasync/latest/userguide/configure-source-location.html)
* [destination location](https://docs.aws.amazon.com/datasync/latest/userguide/create-destination-location.html)

# ddb

* support [atomic counter](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/WorkingWithItems.html)
* local secondary index only can [create at table creation](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/LSI.html#:~:text=Local%20secondary%20indexes%20on%20a%20table%20are%20created%20when%20the%20table%20is%20created)

# eb

* can stop start eb environment with [lambda](https://aws.amazon.com/premiumsupport/knowledge-center/schedule-elastic-beanstalk-stop-restart/) at scheduled time
* doesn't support HTTPS_PROXY

# ebs

* aws [only recommend raid0](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/raid-config.html#:~:text=Creating%20a-,RAID%200,-array%20allows%20you)
* [summary table for different volume types](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html#:~:text=The%20following%20is%20a-,summary,-of%20the%20use%20cases%20and%20characteristics%20of%20SSD)
* [gp2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html#EBSVolumeTypes_gp2)
  * range: 100-16k iops 
  * baseline: base on volume (limited by burst credits)
  * provision: no
* [gp3](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html#gp3-ebs-volume-type)
  * range: 3k-16k iops 
  * baseline: consistent 3k iops
  * provision: 500iops/gb
* [io2, io1](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html)
  * range: 100-32k iops / 32k-64 iops (only available for nitro system )
  * provision: io1: 50iops/gb; io2: 500iops/gb
* io2 block express
  * only support with specific instance ([R5b, X2idn, and X2iedn](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html#:~:text=Note-,io2%20Block%20Express%20volumes%20are%20supported%20with%20R5b%2C%20X2idn%2C%20and%20X2iedn%20instances%20only.,-io2%20Block%20Express))
  * range: 256k iops
  * provision: 1000iops/gb
* [instance store](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/InstanceStorage.html) - temporary block-level storage (physically attach to the host so not an network drive) ([support in specific instance type](https://www.youtube.com/watch?v=7p7p4_4RYxY). it is free)
* the [i/o performance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/storage-optimized-instances.html#storage-instances-diskperf) are limited by ec2 instance type. although you can use raid0 to increase iops but still have a max for that
* queue length on ssd: [1/1000iops](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/benchmark_procedures.html)
* default [block size](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/volume_constraints.html#block_size) is 4kb
* [use case](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonEBS.html#:~:text=The%20following%20is%20a%20summary%20of%20performance%20and%20use%20cases%20for%20each%20volume%20type.) of each volume type
  * gp2, gp3 => boot, dev, test
  * io1, io2 => db
  * st1 (hdd) => large sequential workloads like data / log processing (EMR, ETL, data warehouse)
  * sc1 => save costs
# ec2

* [use cases](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/scenarios-enis.html#creating-dual-homed-instances-with-workloads-roles-on-distinct-subnets) of dual home
  * separate the traffic by role (frontend, backend)
  * ha (move the eni to other instance)
  * security appliance reason
* eni is binding to subnet
* [eni](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html) - when creating the eni, it inherits the public ipv4 address attribute from subnet

# efs

* HA - [regional replication](https://aws.amazon.com/blogs/aws/new-replication-for-amazon-elastic-file-system-efs/) => notice that it means multi az not multi regions
* cross region backup
  * create 1st lambda to backup data from efs to s3 in region a; turn on cross-region replication in s3; create 2nd lambda to restore data from s3 > efs in region b
  * data sync
* backup solution which does not work in cross region
  * [data pipeline](https://docs.aws.amazon.com/efs/latest/ug/alternative-efs-backup.html#backup-steps) => the backup instance cannot mount 2 efs in different regions
  * [efs-to-efs](https://aws.amazon.com/solutions/implementations/efs-to-efs-backup-solution/) => same as data pipeline solution but implemented by lambda function only
  * [aws backup](https://docs.aws.amazon.com/efs/latest/ug/alternative-efs-backup.html#recommended-backup-solutions) => does not support cross region backup
* [dns name](https://docs.aws.amazon.com/efs/latest/ug/mounting-fs-mount-cmd-dns-name.html) - file-system-id.efs.aws-region.amazonaws.com (like cw. us-east-2)
* efs can deliver sub or low single digit millisecond latencies with [> 10gbps through and 500k iops](https://docs.aws.amazon.com/efs/latest/ug/performance.html#:~:text=Amazon%20EFS%20delivers%20more%20than%2010%20gibibytes%20per%20second%20(GiBps)%20of%20throughput%20over%20500%2C000%20IOPS%2C%20and%20sub%2Dmillisecond%20or%20low%20single%20digit%20millisecond%20latencies.)
* launching instance is limited by [the number of vcpu running per account per region running](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-on-demand-instances.html#:~:text=On%2DDemand%20Instance%20limits%20are%20managed%20in%20terms%20of%20the%20number%20of%20virtual%20central%20processing%20units%20(vCPUs))

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
* [support up to 500 nodes and shards](https://aws.amazon.com/elasticache/redis/#:~:text=It%20allows%20you%20to%20scale%20your%20Redis%20Cluster%20environment%20up%20to%20500%20nodes%20and%20500%20shards)

# elb

* classic load balancer only support [at most one subnet per az](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-create-internal-load-balancer.html)

# iam

* set [SAML session tags](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_abac-saml.html) for access control (add attribute to idp metadata)
* [policy to deny access on specific region](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_examples_general.html#example-scp-deny-region) - deny all except the global service
* in console, the [instance profile](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html) are automatically created along with the iam role with the same name
* ArnLike is case-sensitive but support wildwards like * and ?
* [group name limit](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_iam-quotas.html) is 128 characters
* [temporary security credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_control-access_disable-perms.html#:~:text=Temporary%20security%20credentials%20are%20valid%20until%20they%20expire) are valid until they expire

# mTurk

* submit a request to mTurk. outsource manual tasks like taking survey, text recognition, data migration to public

# opsworks

* [setup custom recipe to config the application with other aws services information](https://docs.aws.amazon.com/opsworks/latest/userguide/other-services.html) => [solid example for adding redis cluster connection information to rail application](https://docs.aws.amazon.com/opsworks/latest/userguide/other-services-redis.html)

# org

* org features to enable
  * all
  * consolidated billing
* [scp](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html) is one of the aws organisation feature
  * default is [allow all](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_inheritance_auth.html) => can only use deny list only
  * use allow list => have to remove FullAWSAccess (the default allow all policy)

# other

* [Public Data Sets](https://aws.amazon.com/about-aws/whats-new/2008/12/03/public-data-sets-on-aws-now-available/) - data set for public access. [more details](https://aws.amazon.com/opendata/open-data-sponsorship-program/)
* [fileb://](https://aws.amazon.com/blogs/developer/best-practices-for-local-file-parameters/) is supported in 
  1. kms (key)
  1. ec2 user data (gzip)
  1. s3 (encryption key)
* [govcloud comparson](https://docs.aws.amazon.com/govcloud-us/latest/UserGuide/govcloud-differences.html)
  1. billing and using can be viewed in standard account
  1. only us citizen employees can administer the govcloud
  1. authentication is isolated from amazon.com
  1. network is isolated from other region
* [migrate IBM MQ to Amazon MQ](https://aws.amazon.com/blogs/compute/migrating-from-ibm-mq-to-amazon-mq-using-a-phased-approach/)
* [migrate ibm db2 luw to rds (mysql postgresql)](https://aws.amazon.com/blogs/database/aws-database-migration-service-and-aws-schema-conversion-tool-now-support-ibm-db2-as-a-source/)
* [iot monitor](https://docs.aws.amazon.com/iot/latest/developerguide/monitoring_overview.html) can check whether the rule has been executed

# rds 

* [RMAN restore isn't supported for Amazon RDS for Oracle DB instances](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Appendix.Oracle.CommonDBATasks.RMAN.html#Appendix.Oracle.CommonDBATasks.BackupArchivedLogs) (RMAN is an backup and store tool for oracle db)

# route53

* [health check](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-failover-determining-health-of-endpoints.html) must respond with 2xx or 3xx. support tcp and http
* support [DNSSEC](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/domain-configure-dnssec.html)

# s3

* access control to object c (account c) from request user a (account a) and s3 bucket (account b)
  1. check the [iam role in account a](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-auth-workflow-bucket-operation.html)
  1. check the bucket policy in account b
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
* [reduced redundancy](https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-class-intro.html) is one of the storage class in s3 but not recommend bylaws - may have a chance to lose the object

# secret manager

* set [RotationSchedule](https://aws.amazon.com/blogs/security/how-to-securely-provide-database-credentials-to-lambda-functions-by-using-aws-secrets-manager/) to schedule an auto rotation (to run a custom lambda) the rds password 

# snowball

* tips to increase performance
  * [batch small file](https://docs.aws.amazon.com/snowball/latest/ug/performance.html#:~:text=Batch%20small%20files%20together)
  * multiple copy operations at one time (2 terminals 2 cp command)
  * connect to multiple workstation (1 snowball can connect to multiple workstation)
* step for using snowball
  1. start the snowball
  1. setup workstation by download ova image and import to the vmware
  1. use cp command (something like aws s3api cp) to copy file to snowball
  1. can upload through gui / command line
  1. send the device back to aws. they will import your data to s3
* take at least 1 weeks

# sso

* [permission sets](https://docs.aws.amazon.com/singlesignon/latest/userguide/permissionsetsconcept.html) - 1 permission set has multiple iam policies => associate to user / group
* sso
  * ad (identity provider) -> aws sso -> application (github, dropbox) / aws accounts
  * [sources of identity provider](https://docs.aws.amazon.com/singlesignon/latest/userguide/connectonpremad.html)
    * aws sso
    * ad connector
    * aws managed ad
    * external ad (two way trust)
  * server -> client
  * server = adfs
    * create an app
    * config app sign-in and sign-out url
  * client = integrated website
    * trusted idp
    * config idP's sign-in and sign-out url + cert
  * user login with ad's app endpoint => ad post data to app's sign-in url => app receive and decrypt the data from ad and give permission to user
  * aws iam federation => single account only

# storage gateway

* need to download ova and import the vm to create a endpoint to bridge on-premise and aws
* storage gateway type
  * [volume](https://docs.aws.amazon.com/storagegateway/latest/userguide/StorageGatewayConcepts.html#volume-gateway-concepts) => mount as a disk (iSCSI) => s3 => ebs
    * cached => save some frequently used data to vm
    * stored => completely on s3
  * file => smb / nfs
  * tape => [tape backup software](https://aws.amazon.com/storagegateway/features/#:~:text=Tape%20Gateway%20presents%20an%20iSCSI,your%20tape%2Dbased%20backup%20workflows.)

# support

* [paid support plans](https://aws.amazon.com/premiumsupport/faqs/#:~:text=How%20many%20users%20can%20open%20technical%20support%20cases%3F) allow unlimited number of users to open technical support cases

# swf

* [swf vs step function](https://aws.amazon.com/swf/faqs/#:~:text=When%20should%20I%20use%20Amazon%20SWF%20vs.%20AWS%20Step%20Functions%3F): use step function first. if does not fit => swf
* use case: [processing large product catalogue using Amazon Mechanical Turk](https://aws.amazon.com/swf/faqs/#:~:text=the%20failed%20chunks.-,Use%20case%20%232,-%3A%20Processing%20large%20product)

# vpc

* [5 sg / eni](https://docs.aws.amazon.com/vpc/latest/userguide/amazon-vpc-limits.html#:~:text=cannot%20exceed%201%2C000.-,Security%20groups%20per%20network%20interface,-5)
* for dx, need to [enabled route propagation](https://aws.amazon.com/premiumsupport/knowledge-center/routing-dx-private-virtual-interface/#:~:text=Verify%20that%20you%27ve%20enabled%20route%20propagation%20to%20your%20subnet%20route%20tables)
* [resource arn](https://docs.aws.amazon.com/directconnect/latest/UserGuide/security_iam_service-with-iam.html#:~:text=Direct%20connect%20resource%20ARNs) supported in dx
* [tenancy vpc](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/vpc.html#:~:text=Elastic%20Beanstalk%20doesn%27t%20support%20proxy%20settings%20like%20HTTPS_PROXY%20for%20configuring%20a%20web%20proxy) will determine the instance tenancy by default.
* for vpce, need to ensure the [private dns option](https://docs.aws.amazon.com/vpc/latest/privatelink/vpce-interface.html) is enabled

# worksplace

* use [connection aliases](https://docs.aws.amazon.com/workspaces/latest/adminguide/cross-region-redirection.html) for cross-region workspaces redirection
  * create connection alias
  * share to other account
  * associate with directories in each region
  * setup route53 for failover
  * setup connection string
* [maintenance](https://docs.aws.amazon.com/workspaces/latest/adminguide/workspace-maintenance.html#admin-maintenance) - support regular maintenance windows (eg 15:00-16:00) or manual maintenance but cannot set something like patching on tue 3:00
*  [workspaces application manager](https://docs.aws.amazon.com/wam/latest/adminguide/what_is.html) - package manager to help installing software
* workspaces support [Windows 10 desktop](https://aws.amazon.com/workspaces/faqs/#:~:text=2%20LTS%2C%20or-,Windows%2010%20desktop,-experiences.%20You%20can) but no Windows server
