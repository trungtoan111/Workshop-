---
title : "Store Audit Logs in Amazon S3"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 4.1 </b> "
---

Use the bucket you created earlier (e.g., `network-compliance-logs-<account>-<region>`) and organize prefixes:

- `cloudtrail-logs/` – CloudTrail management/data events  
- `config-logs/` – AWS Config snapshots & configuration history  
- `securityhub-findings/` *(optional)* – if you export findings via EventBridge→Firehose/Lambda  
- `ssm-logs/` *(optional)* – Session Manager session logs

**Bucket settings (recap)**
- **Block Public Access:** ON (all 4)  
- **Versioning:** Enabled  
- **Default Encryption:** SSE-S3 or SSE-KMS (enable *Bucket Key* if KMS)

📸 Upload later:
- `/images/4-1-s3-layout.png` *(Prefixes in the logs bucket)*

---
