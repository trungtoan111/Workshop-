---
title : "Verify Environment Configuration "
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

(In this lab, you need to set up a monitoring environment that will allow AWS Config and AWS Security Hub to continuously evaluate network resources for compliance. This includes enabling core AWS monitoring services, configuring logging, and preparing IAM roles with appropriate permissions. These steps form the foundation for automated compliance checks, audit log collection, and remediation.)

To learn how to create EC2 instances and VPCs with public/private subnets, you can refer to the lab:
  - [About AWS Config](https://000004.awsstudygroup.com/en/)
  - [About AWS Security Hub](https://000003.awsstudygroup.com/en/)

In order to use this solution to automatically monitor and remediate compliance issues, we first need to configure a set of AWS services — including AWS Config, Security Hub, CloudTrail, S3, and IAM — so that they can work together. We will also proceed to create the IAM Roles and policies needed for Lambda functions and Config rules to operate correctly.

### Content
  - [2.1 Prepare Monitoring Environment](2.1-createec2/)

  - [Verify Environment Configuration](2.2-createiamrole/)


