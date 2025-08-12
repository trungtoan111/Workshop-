---
title : "Enable AWS Security Hub"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 3.2.1 </b> "
---

#### Enable AWS Security Hub

**Goal**  
Turn on **AWS Security Hub** in the target Region so security controls and compliance checks can generate **findings**.

> **Prerequisite:** You have permissions to enable Security Hub (`securityhub:*`) and **AWS Config** is already recording in this Region.

---

## Console

1. Open **Security Hub** → **Getting started** → **Enable Security Hub**.  
2. Confirm the **current Region** (repeat in other Regions if needed).  
3. *(Recommended)* In **Settings → General**, enable **Auto-enable new controls** for future standards updates.

📸 Upload later:
- `/images/3-2-sh-enable.png` *(Enable Security Hub screen)*

---

## CLI

```bash
REGION=ap-southeast-1

# Enable Security Hub in this Region
aws securityhub enable-security-hub --region "$REGION"

# Quick check (will list 0 until you enable standards in 3.2.2)
aws securityhub get-enabled-standards --region "$REGION"
```
