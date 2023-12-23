---
title: "AWS Certified Data Analytics - Specialty"
date: 2023-07-31
tags:
- aws
- data
---

## Service-wise Summary

### Athena

**[Compression](https://docs.aws.amazon.com/athena/latest/ug/compression-formats.html)**
- Athena uses different compression methods based on the need:
  - gzip: Used when focusing on reducing the size of data.
  - lzo or snappy: Used when focusing on speed.

**[Workgroup](https://docs.aws.amazon.com/athena/latest/ug/workgroups-setting-control-limits-cloudwatch.html)**
- Access to the workgroup is controlled by IAM policies.
- Workgroups can be temporarily enabled or disabled.
- You can define policies for things like query result location or encryption.
- By turning on 'Override client-side settings', you can enforce the workgroup's settings on users.
- Data usage control can be applied in two ways:
  - Per query: The query stops when the amount of scanned data exceeds a limit.
  - Workgroup-wide: Triggers an alert when the amount of scanned data exceeds a limit, but doesn't enforce the limit.
- Access rights are based on a user's permissions, not on service or EC2 roles.

**Query Timeout**
- DML/DDL queries have a timeout, which can be increased in the [service quota](https://docs.aws.amazon.com/general/latest/gr/athena.html#amazon-athena-limits).

**[Resolving HIVE_METASTORE_ERROR](https://repost.aws/knowledge-center/athena-hive-metastore-error)**
  - Error: "Expected 'xxxx' but '/' found."
    - Reason: This occurs when there's an unsupported character in the partition or column.
    - Fix: Remove the unsupported character.
  - Error: "Storage descriptor is not populated."
    - Reason: This occurs when there's a broken partition.
    - Fix: Use 'msck repair table' to repair the table, or copy the data into a separate folder and run the crawler on that folder.
  - Error: "Payload size exceed."
    - Reason: This occurs when running a Lambda function in a query, as Lambda has a size limit on returned data.
    - Fix: Upload the result to S3, then attach the presigned URL in the response.

### EMR

**[Block Public Access](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-block-public-access.html)**
- EMR has a setting similar to S3's 'Block public access'. 
- This setting functions before EMR creation and removes public access in the security group, except for allowed port ranges.

**[Consistent View](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-plan-consistent-view.html) (Legacy)**
- Previously, AWS used DynamoDB to track read-after-write consistency in S3 objects (using EMRFS).

**EBS Encryption and Custom Software**
- EBS encryption and custom software installation in EMR can be accomplished using a [custom Amazon Linux AMI](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-custom-ami.html).
- Custom software can also be installed using bootstrap actions, similar to the way user-data is used in EC2.

**Integration with DynamoDB**
- EMR has [integration with DynamoDB in Hive](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/EMRforDynamoDB.Tutorial.html).

**[Handling HTTP 503 Slow Down AWSZonS3Exception](https://repost.aws/knowledge-center/emr-s3-503-slow-down)**
- This error is caused by an excessive number of reads to S3.
- It can be fixed by increasing the EMRFS retry limit or adding more prefixes, as the rate limit applies per prefix.

**[HBase](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-hbase-s3.html)**
- HBase is a column-based NoSQL database, ideal for OLTP operations.
- Features include:
  - Data storage in S3.
  - Support for read-replicas (although two masters cannot point to the same HBase path, read-replicas can be recreated).
  - The ability to take snapshots to S3.

**[High Availability](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-plan-ha.html)**
- High availability is currently supported only in a single AZ (support for multiple AZs is coming soon).
- For high availability, it's recommended to create multiple masters and use EMRFS.

**[Presto](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-presto.html)**
- Presto can work with data from multiple sources.

**[S3-Dist-CP](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/UsingEMR_s3distcp.html)**
- S3-Dist-CP is a tool that helps in distributing data copying between EMRFS and HDFS.

**[Sqoop](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-sqoop.html)**
- Sqoop is used to export and import data between relational databases (like RDS) and HDFS.

**[EMR Managed Scaling](https://aws.amazon.com/blogs/big-data/introducing-amazon-emr-managed-scaling-automatically-resize-clusters-to-lower-cost/)**
- EMR Managed Scaling reacts faster than traditional auto scaling.
- It can work with either instance groups or fleets.
- It is automatically configured, so no manual setup is needed.
### Glue

- [Problem with maximum concurrent runs](https://repost.aws/questions/QUXqCW9gvdTUaeWf-xVhLpXg/issue-with-maximum-concurrent-runs-and-job-status
).
  - Once the concurrent limit is reached and a job has already been completed, attempting to rerun the same job will result in a ConcurrentRunsExceededException being triggered.
  - Possible hidden state indicating the job is still running.

- [AWS Glue and Apache Hudi Integration](https://docs.aws.amazon.com/glue/latest/dg/aws-glue-programming-etl-format-hudi.html).

**Crawler**
- Can use S3 locations or data catalog tables as [sources](https://docs.aws.amazon.com/glue/latest/dg/define-crawler.html#crawler-source-type).
- When S3 location is chosen, it automatically creates or updates the schema and table.
- When a data catalog table is chosen, it does not [create a new table](https://docs.aws.amazon.com/glue/latest/dg/tables-described.html#update-manual-tables).
- Choices are available for handling schema updates or deletions.

**ETL Job**
- In [ETL job](https://docs.aws.amazon.com/glue/latest/dg/update-from-job.html), you have the option to update the schema.

**[Debugging OOM Exceptions](https://docs.aws.amazon.com/glue/latest/dg/monitor-profile-debug-oom-abnormalities.html):**

**Driver**
- Problem: The driver creates a task for each file in an S3 prefix. When there are too many files in one location, it can lead to an OOM error due to the large number of tasks.
- Solution: Use '[groupFiles: inPartition](https://docs.aws.amazon.com/glue/latest/dg/grouping-input-files.html)' to load multiple files in one task, which reduces the number of tasks and prevents OOM.

**Executor**
- Problem: When selecting data from MySQL, the default action is to load all rows. This can cause an OOM error if there are too many rows.
- Solution: Use a dynamic frame to limit the number of rows selected at once, which prevents OOM.

**[Excluding S3 Storage Class in ETL Job](https://docs.aws.amazon.com/glue/latest/dg/aws-glue-programming-etl-storage-classes.html)**
- The S3 storage class can be excluded by setting it in the data catalog property or in the dynamic frame.

**[FindMatches](https://docs.aws.amazon.com/glue/latest/dg/machine-learning.html)**
- Uses machine learning to find records from different tables that refer to the same object (such as a customer or product).

**PostAction**
- Can be used to deduplicate records before ingesting data into Redshift.

### Kinesis Stream

**[Resharding](https://docs.aws.amazon.com/streams/latest/dev/kinesis-using-sdk-java-after-resharding.html)**
- This involves splitting and merging shards.
- For instance, during a split, shard1 can be divided into shard10 and shard11. Here, shard1 becomes the parent, while shard10 and shard11 become the children.
- After resharding, the parent shard (shard1 in this case) becomes read-only but remains available. For ordered data reading, the parent should be consumed first, followed by the children. This process needs to be managed in the SDK.

**[Enhanced Fanout](https://aws.amazon.com/vi/blogs/architecture/field-notes-how-to-scale-opentravel-messaging-architecture-with-amazon-kinesis-data-streams/)**
- Kinesis limits throughput by shard (1MB for write, 2MB for read). Enhanced fanout can increase read throughput.
- This process uses a Lambda function to read from the stream and Lambda replicas to distribute the data to more destinations.

**Kinesis Stream as a Database**
- The data is stored until it expires. The expiry period can be set from 1 day to 1 year.
- The data is not removed when read, meaning it can be consumed multiple times. 

**Stateless Nature**
- Consumers need to remember their read position to prevent re-reading data. This is automatically handled in the KCL, which creates a DDB table to manage consumer memberships and track their states.
- If using the SDK, developers must implement the checkpoint by tracking the number of records read or by time to achieve the same effect.

**Using KCL**
- A [DDB table](https://docs.aws.amazon.com/streams/latest/dev/shared-throughput-kcl-consumers.html#shared-throughput-kcl-consumers-leasetable) is created to maintain the consumer state. 
- If the [iterator expires](https://docs.aws.amazon.com/streams/latest/dev/troubleshooting-consumers.html#shard-iterator-expires-unexpectedly) unexpectedly, it could be due to the DDB not having enough write capacity.

**[Handling Duplicate Records](https://docs.aws.amazon.com/streams/latest/dev/kinesis-record-processor-duplicates.html)**
- Sequence numbers can be used to identify duplicates. 
- Retries are common in streams and can occur due to network issues or unexpected consumer exits, which can lead to the checkpoint not being saved.

### Kinesis Firehose

- Data can be delivered to storage (S3) or databases (DDB, Redshift, RDS, OpenSearch).
- The minimum batch time limit is 1 minute, which may not be ideal for real-time processing.
- A Lambda function can be used to transform data before it's delivered or to send alerts.

### Kinesis Analytics

- A CSV file in S3 can be attached as [reference data](https://docs.aws.amazon.com/kinesisanalytics/latest/dev/app-add-reference-data.html) to join with the stream.
- The data can be output to [Lambda](https://docs.aws.amazon.com/kinesisanalytics/latest/dev/how-it-works-output-lambda.html).
- [RANDOM_CUT_FOREST](https://docs.aws.amazon.com/kinesisanalytics/latest/sqlref/sqlrf-random-cut-forest.html) can be used for anomaly detection.
- Data can be pre-processed with a [Lambda](https://docs.aws.amazon.com/kinesisanalytics/latest/dev/lambda-preprocessing.html) function.

### OpenSearch

**[Issues leading to High JVM Pressure](https://repost.aws/knowledge-center/opensearch-high-jvm-memory-pressure)**
- Having too many divisions of an index, also known as shards.
- Uneven distribution of shards.
- Running summaries on text fields or wildcards.
- Combining large data tables.
- An overload of requests.

**[UltraWarm](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/ultrawarm.html) Feature**
- A cost-effective, read-only feature.
- The index state management tool can automate moving data to UltraWarm.
- To fine-tune performance, adjust the 'max_num_segments' value in 'force_merge'. A larger 'max_num_segments' makes the process faster, but can cause delays after indexing.

### Quicksight

**Setting Up [Row-Level Security](https://docs.aws.amazon.com/quicksight/latest/user/restrict-access-to-a-data-set-using-row-level-security.html) on a Dataset**
- Attach a CSV file or a user-based query rule to the dataset.
- The file or query includes "user" or "group" and filtering conditions (similar to SQL's WHERE clause: WHERE variable=value).
- The default setting is to restrict access; if there's no entry, there's no visibility.
- Tag-based rules can be used in API calls to create embedded URLs for guest users.

**Implementing Column Level Security**
- Choose the columns that specific users or groups can view.
- To retrieve data from Athena, establish a connection to Athena and register the S3 bucket in Quicksight.

**Features of the Paid Version**
- [Isolated namespaces](https://docs.aws.amazon.com/quicksight/latest/user/namespaces.html) that readers can access.
- Authentication via Active Directory.

### Redshift

**[DBlink](https://aws.amazon.com/blogs/big-data/join-amazon-redshift-and-amazon-rds-postgresql-with-dblink/)**
- An add-on to PostgreSQL that enables data queries directly from Redshift. It operates in read-only mode.

**[Audit Log](https://docs.aws.amazon.com/redshift/latest/mgmt/db-auditing.html)**
- Can be activated as needed.

**[Column Level Access Control](https://docs.aws.amazon.com/redshift/latest/dg/r_GRANT-usage-notes.html#r_GRANT-usage-notes-clp)**
- Access can be provided through the GRANT SELECT(COL1, COL2,...) command.

**[Query Monitoring Rule](https://docs.aws.amazon.com/redshift/latest/dg/cm-c-wlm-query-monitoring-rules.html)**
A rule includes:
- A rule name.
- One or more conditions.
- An action to be taken (stop, log, hop, change priority).

**[Short Query Acceleration](https://docs.aws.amazon.com/redshift/latest/dg/wlm-short-query-acceleration.html)**
- In the workload management settings, this feature can prioritize faster queries.

**Spectrum**
- This feature is not compatible with Glacier data.

### S3

**[S3 Select](https://docs.aws.amazon.com/AmazonS3/latest/userguide/selecting-content-from-objects.html)**
- For Glacier, it functions only with instant retrieval.
- Intelligent Tiering and Reduced Redundancy Storage are not compatible.
- Data output can be formatted as CSV or JSON.
- Data input can be formatted as CSV or JSON. Compression is supported, but only gzip or bzip formats are allowed. For columnar formats like Parquet, only gzip or snappy formats are allowed.
- [S3 Object Lock](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html) can help prevent Amazon S3 objects from being deleted or overwritten for a fixed amount of time or indefinitely. 

### Lake Formation

- To [maintain compatibility](https://docs.aws.amazon.com/lake-formation/latest/dg/upgrade-glue-lake-formation-background.html), "super" permission is given to IAMAllowedPrincipals. This means that if the IAM policy permits the data catalog, it will bypass the Lake Formation's permission control.

Here are the steps to use Lake Formation:

- Take away the IAMAllowedPrincipals' rights in Data Lake permissions, under Administrative roles and tasks for Database Creators.
- In the data catalog settings, deselect 'Use only IAM access control for new databases' and 'Use only IAM access control for new tables in new databases'.
- Register the S3 in the Data Lake location.
- Assign permissions in Data Lake permissions. If 'select' permission is given, it will also allow the S3 permission.

## Topic-wise Summary

### Encryption

**[Quicksight](https://docs.aws.amazon.com/quicksight/latest/user/key-management.html)**: Employs AWS managed keys or KMS-CMK for data encryption.
  
**[Redshift](https://docs.aws.amazon.com/redshift/latest/mgmt/working-with-db-encryption.html)**: Utilizes CloudHSM, on-premise HSM, or KMS for data encryption.
  
**EMR**: 
- For EMRFS: 
  - Server side: Uses either AWS managed keys or customer-managed KMS keys.
  - Client side: Employs customer-managed KMS keys or custom KMS keys.
- For instance storage/EBS: Prefers EBS when possible. If not, it falls back to LUKS encryption.

### Access Control

**Concepts**

**Row-Level Security**: 
- Conceals rows by filtering data based on a certain value (e.g., using the SQL command: SELECT * FROM table WHERE foo=bar).

**Column-Level Security**: 
- Hides specific columns (these columns will not be included in a SELECT statement).

**Athena**: 
- Access is managed through user/group permissions.

**EMR**: 
- Access is managed via an EC2 role.
- For EMRFS, an IAM role is defined in the security configuration (this role is assumed by the EC2 role).

**[Redshift](https://docs.aws.amazon.com/redshift/latest/dg/copy-parameters-authorization.html)**: 
- Access is managed via a service role.
- Usage permissions are granted to IAM roles (similar to MySQL).

**[Quicksight](https://docs.aws.amazon.com/quicksight/latest/user/identity.html)**: 
- Implements both row-level and column-level security on datasets.
- Authentication methods:
  - IAM/SAML2/MFA are available for free.
  - Active Directory is available with the Enterprise edition.

**Lake Formation**: 
- Implements both row-level and column-level security on the data catalog.

**Best Practices**:

**Redshift**: 
- To use data in Lake Formation, grant SELECT permission to the Redshift role.
- To allow a user to use a table in Redshift, use GRANT USAGE in Redshift.

**EMRFS**: 
- S3 access should be granted through an IAM role, not an EC2 role. 
- The EC2 role assumes the IAM role to retrieve data from S3.


## DMS

- [Metrics for monitoring database migration](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Monitoring.html#CHAP_Monitoring.Metrics).
- CDCLatencySource and CDCLatencyTarget indicate latency between the source and target in replication.