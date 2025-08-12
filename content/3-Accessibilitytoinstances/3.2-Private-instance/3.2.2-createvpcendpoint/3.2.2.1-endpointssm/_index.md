---
title : "PCI-DSS"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 3.2.2.1 </b> "
---

#### Enable SOC 2 Standard in AWS Security Hub

In this step, you will **enable the SOC 2 security standard** in AWS Security Hub and review key controls relevant to **Network Compliance and Audit Automation** (encryption, logging, and network restrictions). This ensures findings are centralized and mapped to your AWS Config rules.

---

### Steps

1. Open **AWS Security Hub** in the AWS Console.  
   + Go to **Security standards**.  
   + Click **SOC 2** → **Enable standard** for your target Region.

![Enable SOC 2]({{ "images/3.2.2.1-soc2-enable.png" | relURL }})


2. Review and (optionally) tune **SOC 2 controls** commonly used in this workshop:  
   + **CloudTrail** enabled & **Log file validation** enabled.  
   + **S3 encryption at rest** and **Block Public Access** enabled.  
   + **EBS encryption by default**.  
   + **VPC Flow Logs** enabled.  
   + **Least-privilege IAM** (no root access keys, access key rotation).

> Tip: These align with the AWS Config rules you added earlier (`cloud-trail-enabled`, `cloud-trail-log-file-validation-enabled`, `s3-bucket-server-side-encryption-enabled`, `ec2-ebs-encryption-by-default`, `vpc-flow-logs-enabled`, etc.).

![Review SOC 2 Controls](/images/3.2.2.1-soc2-controls.png)

3. Validate **SOC 2 findings** appear:  
   + Open **Findings** in Security Hub.  
   + Filter by **Standards = SOC 2** (and, if needed, **Product name = AWS Security Hub**).  
   + Confirm new findings populate as controls evaluate.  
   + Click a finding to see the **affected resource** and **remediation** guidance.

![Validate SOC 2 Findings](/images/3.2.2.1-soc2-findings.png)

---

**Outcome:** SOC 2 standard is enabled; related controls are evaluating and producing findings that you can use to drive **automated remediation** and **audit reporting** in later steps.


