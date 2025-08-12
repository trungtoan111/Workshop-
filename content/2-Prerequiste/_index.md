---
title : "Configure Monitoring Environment "
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

(In this lab, you will set up a monitoring environment that enables AWS Config and AWS Security Hub to continuously evaluate AWS network resources for compliance with frameworks such as SOC 2, PCI-DSS, and HIPAA. This includes enabling core AWS monitoring services, configuring centralized logging, and preparing IAM roles with the appropriate permissions. These steps form the foundation for automated compliance checks, audit log collection, and remediation.)

To learn more about the services used in this step, you can refer to:
  - [About AWS Config](https://000004.awsstudygroup.com/en/)
  - [About AWS Security Hub](https://000003.awsstudygroup.com/en/)
    
In order to automatically monitor and remediate compliance issues, we first need to configure a set of AWS services — including AWS Config, Security Hub, CloudTrail, Amazon S3, and AWS IAM — so they work together seamlessly. We will also create the IAM roles and policies required for Lambda functions, AWS Config rules, and reporting services to operate securely and effectively.


