---
title : "Integrate Security Hub with AWS Config"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 3.2. </b> "
---

**Goal**  
Enable **AWS Security Hub** and integrate it with **AWS Config** so compliance/control evaluations (FSBP, CIS, PCI DSS) surface as unified **findings**. Many Security Hub controls are implemented using **AWS Config managed rules**, so making sure both services are enabled and connected gives you a single view for alerts and evidence.

### Architecture

![Security Hub + Config integration](/images/3-2-sh-arch.png)

> **Prerequisite:** AWS Config recording is already enabled in this Region (from Section 2).

---

## What you will do

1) **Enable Security Hub** in the Region(s) you monitor.  
2) **Enable standards** (e.g., AWS Foundational Security Best Practices, CIS, PCI DSS).  
3) **Verify the integration with AWS Config** (usually enabled by default).  
4) *(Optional)* **Aggregate findings across Regions** into a home Region.  
5) **Validate** controls & findings are flowing.

---

## A) Enable Security Hub

**Console**
1. Open **Security Hub** → **Getting started** → **Enable Security Hub**.  
2. Choose the **current Region** (repeat in others as needed).

📸 Upload later:
- `/images/3-2-sh-enable.png` *(Enable Security Hub)*

**CLI**
```bash
REGION=ap-southeast-1
aws securityhub enable-security-hub --region "$REGION"
```

## B) Enable standards (FSBP / CIS / PCI DSS)
**Console**

1. Security Hub →**Standards** → **Enable standard**.
2. Enable:

**AWS Foundational Security Best Practices (FSBP)**
**CIS AWS Foundations Benchmark**
**PCI DSS (if applicable)**

📸 Upload later:

/images/3-2-sh-standards.png (Standards enabled)

**CLI (discover ARNs, then enable)**
```bash
# Discover available standards and versions in your Region
aws securityhub describe-standards --region "$REGION"

# Example: set ARNs from your Region's output (replace if versions differ)
FSBP_ARN="arn:aws:securityhub:${REGION}::standards/aws-foundational-security-best-practices/v/1.0.0"
CIS_ARN="arn:aws:securityhub:${REGION}::standards/cis-aws-foundations-benchmark/v/1.4.0"
PCI_ARN="arn:aws:securityhub:${REGION}::standards/pci-dss/v/3.2.1"

aws securityhub batch-enable-standards \
  --region "$REGION" \
  --standards-subscription-requests \
  StandardsArn="$FSBP_ARN" \
  StandardsArn="$CIS_ARN" \
  StandardsArn="$PCI_ARN"
```

## C) Verify integration with AWS Config
**Console**

Security Hub → **Integrations** → **AWS services.**

Find **AWS Config** → ensure it shows **Enabled**
(Many Security Hub controls run on top of Config managed rules.)

📸 Upload later:

/images/3-2-sh-integrations.png (Integrations – AWS Config: Enabled)

## D) (Optional) Aggregate findings across Regions
**Console**

Security Hub → **Settings** → **Finding aggregation** → **Linking mode**: **All regions** → **Save**.

**CLI**
```bash

HOME_REGION="$REGION"   # choose a "home" Region for the rollup view
aws securityhub create-finding-aggregator \
  --region "$HOME_REGION" \
  --linking-mode ALL_REGIONS
📸 Upload later:

/images/3-2-sh-aggregator.png (Finding aggregation enabled)
```

## E) Validate controls & findings
**Console**

**Standards → Controls:** status moves from Evaluating → Passed/Failed.

**Findings:** check for ACTIVE findings.

📸 Upload later:

/images/3-2-sh-controls.png (Controls view)

/images/3-2-sh-findings.png (Findings list)

**CLI** (quick checks)
```bash
# List enabled standards
aws securityhub get-enabled-standards --region "$REGION"

# Fetch a few active findings
aws securityhub get-findings \
  --region "$REGION" \
  --filters '{"RecordState":[{"Comparison":"EQUALS","Value":"ACTIVE"}]}' \
  --max-results 5
```
