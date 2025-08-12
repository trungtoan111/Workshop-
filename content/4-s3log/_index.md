---
title : "Manage and Analyze Audit Logs"
date : "`r Sys.Date()`"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---


**Goal**  
Centralize and analyze **audit & compliance logs** from **CloudTrail** and **AWS Config** in **Amazon S3**, query them with **Athena**, and prepare datasets for **QuickSight** dashboards.

**By the end you will**
- Store logs in a secure, versioned **S3** bucket (from Section 2).  
- Enable **CloudTrail (multi-Region + validation)** to capture all API activity.  
- Catalog log data with **AWS Glue** and query with **Amazon Athena**.  
- (Optional) Prepare **QuickSight** datasets and enable **Session Manager** session logging for audit evidence.

### Architecture
![Logs architecture – S3 + CloudTrail + Config + Athena]( /images/4-arch-logs.png )



