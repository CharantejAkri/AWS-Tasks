# Elastic Beanstalk Hybrid App with EFS & Serverless Processing

## Overview

This project demonstrates a **production-grade hybrid AWS architecture** combining:

* Platform-as-a-Service (**Elastic Beanstalk**)
* Shared storage (**Amazon EFS**)
* Event-driven processing (**AWS Lambda**)
* Secure uploads (**Pre-Signed URLs**)

It simulates a **real-world scalable document processing system** where users upload files, which are processed asynchronously and made available across multiple instances.

---

## Architecture

```text
User (Browser)
   ↓ HTTPS
Application Load Balancer (ALB)
   ↓
Elastic Beanstalk (Django App)
   ├── EC2 Instance 1
   └── EC2 Instance 2
          ↕
      Amazon EFS (/var/app/shared/)
          ↑
     AWS Lambda (Processor)
          ↑
S3 Upload (Pre-Signed URL)
          ↓
Processed Output → EFS
```

---

## Key Features

* 🔹 Highly available Django application using Elastic Beanstalk
* 🔹 Shared file system across instances using EFS
* 🔹 Secure file upload using pre-signed URLs
* 🔹 Event-driven processing using Lambda (S3 trigger)
* 🔹 Auto Scaling enabled for load handling
* 🔹 Lifecycle policies and backup strategy

---

## Tech Stack

* **Compute:** EC2 (via Elastic Beanstalk)
* **Storage:** S3, EFS
* **Serverless:** Lambda
* **Networking:** ALB
* **Monitoring:** CloudWatch
* **Language:** Python (Django, Boto3)

---

## How It Works

1. User requests a **pre-signed upload URL** from Django app
2. File is uploaded directly to S3
3. S3 triggers a Lambda function
4. Lambda processes the file
5. Output is stored in **EFS (shared storage)**
6. All EC2 instances can access processed files

---

## Deployment Steps (High-Level)

### 1. Setup S3

* Create upload and processed buckets
* Enable versioning and encryption

### 2. Setup EFS

* Create file system
* Attach mount targets
* Allow NFS access

### 3. Deploy Elastic Beanstalk

* Python 3.11 environment
* Configure environment variables
* Mount EFS using `.ebextensions`

### 4. Develop Django App

* API to generate pre-signed URLs
* Integrate with S3 using Boto3

### 5. Setup Lambda

* Triggered by S3 uploads
* Processes files and writes to EFS

### 6. Configure Auto Scaling

* Enable scaling policies
* Validate under load

---

## Validation Checklist

* ✔ Elastic Beanstalk environment is healthy
* ✔ File upload via pre-signed URL works
* ✔ Lambda triggers successfully
* ✔ Processed file stored in EFS
* ✔ Accessible across instances
* ✔ Auto Scaling works

---

## Cost Note

# This project is **not fully free-tier eligible**

**Paid components include:**

* Application Load Balancer (ALB)
* Amazon EFS
* Multiple EC2 instances

# A simplified free-tier version can be built using:

* Single EC2 instance
* S3 instead of EFS
* No ALB

---

## Key Learnings

* Elastic Beanstalk customization using `.ebextensions`
* Shared storage with EFS in distributed systems
* Event-driven architecture (S3 → Lambda)
* Secure uploads using pre-signed URLs
* Auto Scaling in production environments

---

## Cleanup

To avoid charges:

* Terminate Elastic Beanstalk environment
* Delete S3 buckets
* Delete Lambda function
* Delete EFS
* Remove backup plans

---

## Use Cases

* Document processing systems
* Media pipelines
* Multi-instance file sharing apps
* Enterprise backend systems

---
