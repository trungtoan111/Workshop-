---
title : "Enable AWS Security Hub"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 3.2.1 </b> "
---

#### Enable DNS Hostnames on VPC

> **Goal:** Enable **DNS hostnames** on your VPC to allow private service endpoints and compliance service integrations.

1. **Open VPC Service**
   + Go to [VPC console](https://console.aws.amazon.com/vpc/home).
   + Click **Your VPCs** in the left panel.
   + Select your **Lab VPC** (created in the prerequisite step).
   + Click **Actions** → **Edit DNS hostnames**.

   📸 *Upload screenshot here:*  
   `![VPC Console – Your VPCs](/images/3.connect/your-vpcs.png)`

2. **Enable DNS Hostnames**
   + On the **Edit DNS hostnames** page:
     - Select **Enable**.
     - Click **Save changes**.

   📸 *Upload screenshot here:*  
   `![Enable DNS Hostnames](/images/3.connect/enable-dns-hostnames.png)`

---

✅ **Tip:** This setting is required so AWS-managed private endpoints (e.g., Config, Security Hub) can resolve internal service names inside your VPC.
