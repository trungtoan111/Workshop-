---
title : "Create Athena Tables for Log Analysis"
date : "`r Sys.Date()`"
weight : 3
chapter : false
pre : " <b> 4.3 </b> "
---


We’ll use **AWS Glue Crawlers** to infer schemas automatically (simpler & less error-prone than hand-writing DDL).

**Console – Glue Crawler (CloudTrail)**

**1. AWS Glue → Crawlers → Create crawler**

**2. 2Data source:** S3 → s3://<logs-bucket>/cloudtrail-logs/

**3. IAM role:** create/use a Glue role with S3 read access to the logs bucket

**4. Target database:** create compliance_logs

**5. Schedule:** On demand (you can re-run to pick new partitions)

**6. Finish → Run crawler**

**Console – Glue Crawler (AWS Config)**

Repeat the steps with data source s3://<logs-bucket>/config-logs/ (snapshots & history)

📸 Upload later:

/images/4-3-glue-crawler-ct.png (Crawler for CloudTrail)

/images/4-3-glue-crawler-config.png (Crawler for AWS Config)

/images/4-3-athena-db-tables.png (Database & tables created)

**Athena – set query result location**

In **Athena → Settings**, set results to s3://<logs-bucket>/athena-results/

## Sample Athena queries
## A) Recent security-group changes (CloudTrail)
```bash 
sql
Sao chép
Chỉnh sửa
SELECT
  from_iso8601_timestamp(eventtime) AS event_time,
  useridentity.principalid        AS principal_id,
  useridentity.arn                AS principal_arn,
  eventname,
  requestparameters.groupId       AS security_group_id
FROM compliance_logs.<your_cloudtrail_table>
WHERE eventsource = 'ec2.amazonaws.com'
  AND eventname IN ('AuthorizeSecurityGroupIngress','RevokeSecurityGroupIngress')
  AND from_iso8601_timestamp(eventtime) >= date_add('day', -7, current_timestamp)
ORDER BY event_time DESC
LIMIT 100;
```
## B) Non-encrypted S3 buckets detected (via Config snapshot/history)
(Example using a crawler-generated table for Config history)
```bash
sql
Sao chép
Chỉnh sửa
SELECT
  ci.accountid,
  ci.resourceid     AS bucket_name,
  ci.awsregion      AS region,
  ci.configuration.bucketEncryption IS NULL AS no_encryption
FROM compliance_logs.<your_config_table> AS ci
WHERE ci.resourcetype = 'AWS::S3::Bucket'
  AND (ci.configuration.bucketEncryption IS NULL OR cardinality(ci.configuration.bucketEncryption.rules) = 0)
ORDER BY ci.accountid, bucket_name;
```
## C) Who disabled CloudTrail log file validation? (CloudTrail)
```bash
sql
Sao chép
Chỉnh sửa
SELECT
  from_iso8601_timestamp(eventtime) AS event_time,
  useridentity.arn,
  eventname,
  requestparameters
FROM compliance_logs.<your_cloudtrail_table>
WHERE eventsource = 'cloudtrail.amazonaws.com'
  AND eventname IN ('UpdateTrail','StopLogging')
ORDER BY event_time DESC
LIMIT 50;
```
