---
title : " Verify AWS Config and Rules Status"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 2.1 </b> "
---

Open the AWS Config Dashboard

In the AWS Management Console → Search for AWS Config → Open the Dashboard.

Check data collection status

The Configuration Items Recorded section must display the latest number of records.

Ensure that the Configuration Recorder is not showing any permission errors (e.g., Insufficient Permissions).

Check the compliance status of the Rules

In the Compliance status section, verify:

The Rules you created in step 2.1.3 appear in the list.

The Detective compliance column shows the status:

Compliant = meets the requirement

Noncompliant = does not meet the requirement (needs remediation in later steps)

