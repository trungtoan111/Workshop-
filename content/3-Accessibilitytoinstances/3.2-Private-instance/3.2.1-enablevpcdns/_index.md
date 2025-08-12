---
title : "Enable AWS Security Hub"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 3.2.1 </b> "
---

#### Enable AWS Security Hub

1. Mở **AWS Security Hub** trong AWS Console.  
   + Nhấn **Enable Security Hub** cho Region bạn đang làm lab.  
   + (Khuyến nghị) Bật **Auto-enable new controls**.

![Enable Security Hub](/images/3.connect/sh-enable-service.png)

2. Bật các **Security standards** cần thiết cho lab (có thể chọn nhiều):  
   + **SOC 2** – kiểm soát về logging, encryption, IAM.  
   + **PCI-DSS** – các kiểm soát về segmentation, firewall rules, access logging.  
   + **HIPAA** – yêu cầu mã hóa in-transit & at-rest, audit logging chi tiết.

![Enable Standards](/images/3.connect/sh-enable-standards.png)

3. Tích hợp **Security Hub** với **AWS Config** để gom findings về một nơi:  
   + Vào **Security Hub → Settings → Integrations → AWS services**.  
   + Tìm **AWS Config** và bấm **Enable integration**.  
   + Mở **Findings** và lọc theo `Product name = AWS Config` để xác minh dữ liệu đã đổ về.

![Integrate with AWS Config](/images/3.connect/sh-config-integration.png)
