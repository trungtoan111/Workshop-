---
title   : "Create AWS Config Rules"
date    : "`r Sys.Date()`"
weight  : 3
chapter : false
pre     : " <b> 2.1.3 </b> "
---

**Goal**  
Add **AWS Config managed rules** to continuously evaluate your **network/security posture** (and feed Security Hub controls). Start with high-value rules, then expand as needed.

> **Prerequisites**
> - **AWS Config** is enabled and recording (2.1.2).
> - **S3 audit bucket** exists for snapshots/evaluations (2.1.1).

---

## Recommended managed rules (network-focused)

Begin with these (you can add more later):

- **S3_BUCKET_SERVER_SIDE_ENCRYPTION_ENABLED** – default encryption on S3 buckets  
- **S3_BUCKET_PUBLIC_READ_PROHIBITED** / **S3_BUCKET_PUBLIC_WRITE_PROHIBITED** – block public buckets  
- **S3_BUCKET_SSL_REQUESTS_ONLY** – require HTTPS  
- **RESTRICTED_SSH** – no 0.0.0.0/0 on port 22  
- **RESTRICTED_INCOMING_TRAFFIC** (or **RESTRICTED_COMMON_PORTS**) – common risky ports not open  
- **VPC_FLOW_LOGS_ENABLED** – VPC/Subnet/ENI flow logs must be enabled  
- **EIP_ATTACHED** – ensure Elastic IPs are attached  
- **ELBV2_ALB_HTTP_TO_HTTPS_REDIRECTION_CHECK** – redirect ALB HTTP → HTTPS  
- **ELB_TLS_SECURITY_POLICY_CHECK** – load balancer uses modern TLS policy

> Names above are **managed rule identifiers** you can select in Console. For CLI, use the **SourceIdentifier** shown.

📸 Upload later:
- `/images/2-1-3-rules-list.png` *(Rules list with status)*
- `/images/2-1-3-rule-create.png` *(Add managed rule dialog)*
- `/images/2-1-3-rule-details.png` *(Rule details & parameters)*

---

## A) Create rules (Console)

1. Open **AWS Config → Rules → Add rule**.  
2. Search each rule name above (e.g., **RESTRICTED_SSH**).  
3. For rules with **parameters** (e.g., authorized ports, specific VPC IDs), set them to match your environment.  
4. Click **Next → Add rule**. Repeat for all.

> **Tip:** Tag rules with `Compliance=Network` so you can slice reports later.

---

## B) Create rules (CLI – examples)

```bash
REGION="ap-southeast-1"

# 1) S3 default encryption
aws configservice put-config-rule --region "$REGION" --config-rule '{
  "ConfigRuleName": "s3-bucket-encryption-required",
  "Description": "All S3 buckets must have default encryption enabled.",
  "Scope": { "ComplianceResourceTypes": ["AWS::S3::Bucket"] },
  "Source": {
    "Owner": "AWS",
    "SourceIdentifier": "S3_BUCKET_SERVER_SIDE_ENCRYPTION_ENABLED"
  }
}'

# 2) SSH not open to the world
aws configservice put-config-rule --region "$REGION" --config-rule '{
  "ConfigRuleName": "restricted-ssh",
  "Description": "Disallow 0.0.0.0/0 on port 22.",
  "Source": {
    "Owner": "AWS",
    "SourceIdentifier": "RESTRICTED_SSH"
  }
}'

# 3) VPC Flow Logs enabled
aws configservice put-config-rule --region "$REGION" --config-rule '{
  "ConfigRuleName": "vpc-flow-logs-enabled",
  "Description": "VPC/Subnet/ENI must have Flow Logs enabled.",
  "Source": {
    "Owner": "AWS",
    "SourceIdentifier": "VPC_FLOW_LOGS_ENABLED"
  }
}'

# 4) ALB must redirect HTTP->HTTPS (ELBv2)
aws configservice put-config-rule --region "$REGION" --config-rule '{
  "ConfigRuleName": "alb-http-to-https-redirect",
  "Description": "ALB listeners should redirect HTTP to HTTPS.",
  "Source": {
    "Owner": "AWS",
    "SourceIdentifier": "ELBV2_ALB_HTTP_TO_HTTPS_REDIRECTION_CHECK"
  }
}'
```

Note: If a rule needs parameters (e.g., allowed ports, target VPCs), add "InputParameters": "{\"paramName\":\"value\"}" to the JSON.

## C) (Optional) Deploy a Network Baseline Conformance Pack
Use a single YAML to deploy a bundle of rules consistently across accounts/Regions.

**Console**

**AWS Config → Conformance packs → Deploy conformance pack**

Paste the YAML below → name it network-baseline-pack → Deploy

**YAML (sample)**
```yaml
Parameters:
  AllowedSshCidr:
    Type: String
    Default: "10.0.0.0/8"
  RequireHttpsOnly:
    Type: String
    Default: "true"

Resources:
  S3Encryption:
    Type: "AWS::Config::ConfigRule"
    Properties:
      ConfigRuleName: "s3-bucket-encryption-required"
      Source:
        Owner: "AWS"
        SourceIdentifier: "S3_BUCKET_SERVER_SIDE_ENCRYPTION_ENABLED"

  S3PublicReadProhibited:
    Type: "AWS::Config::ConfigRule"
    Properties:
      ConfigRuleName: "s3-public-read-prohibited"
      Source:
        Owner: "AWS"
        SourceIdentifier: "S3_BUCKET_PUBLIC_READ_PROHIBITED"

  S3PublicWriteProhibited:
    Type: "AWS::Config::ConfigRule"
    Properties:
      ConfigRuleName: "s3-public-write-prohibited"
      Source:
        Owner: "AWS"
        SourceIdentifier: "S3_BUCKET_PUBLIC_WRITE_PROHIBITED"

  S3HttpsOnly:
    Type: "AWS::Config::ConfigRule"
    Properties:
      ConfigRuleName: "s3-https-only"
      Source:
        Owner: "AWS"
        SourceIdentifier: "S3_BUCKET_SSL_REQUESTS_ONLY"

  RestrictedSsh:
    Type: "AWS::Config::ConfigRule"
    Properties:
      ConfigRuleName: "restricted-ssh"
      Source:
        Owner: "AWS"
        SourceIdentifier: "RESTRICTED_SSH"

  VpcFlowLogsEnabled:
    Type: "AWS::Config::ConfigRule"
    Properties:
      ConfigRuleName: "vpc-flow-logs-enabled"
      Source:
        Owner: "AWS"
        SourceIdentifier: "VPC_FLOW_LOGS_ENABLED"

  ElbTlsPolicy:
    Type: "AWS::Config::ConfigRule"
    Properties:
      ConfigRuleName: "elb-tls-policy-check"
      Source:
        Owner: "AWS"
        SourceIdentifier: "ELB_TLS_SECURITY_POLICY_CHECK"

  AlbHttpToHttps:
    Type: "AWS::Config::ConfigRule"
    Properties:
      ConfigRuleName: "alb-http-to-https-redirect"
      Source:
        Owner: "AWS"
        SourceIdentifier: "ELBV2_ALB_HTTP_TO_HTTPS_REDIRECTION_CHECK"
```
📸 Upload later:

/images/2-1-3-pack-deploy.png (Deploy conformance pack)

/images/2-1-3-pack-compliance.png (Pack compliance summary)

## D) Validate
**AWS Config → Rules:** statuses move from Evaluating → Compliant/Noncompliant.

**Evaluations in S3:** objects appear under config-logs/AWSLogs/<ACCOUNT_ID>/Config/<region>/.

If Security Hub standards are on (3.2.2), related controls will surface findings.

📸 Upload later:

/images/2-1-3-rule-compliance.png (Rule compliance view)

/images/2-1-3-s3-evaluations.png (S3 keys for evaluations)

## E) Good practices
Start detect-only, then enable auto-remediation (Section 5) for rules you trust.

Tag rules with Compliance=Network, Owner=<team>; use this in reports and filters.

In multi-account setups, deploy via Conformance Packs or IaC (CloudFormation/Terraform) for consistency.
