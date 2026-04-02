# Highly Available Web Application on AWS
**Objective**

The objective of this project is to design and deploy a highly available, scalable, and fault-tolerant web application on AWS.

This includes:

Distributing infrastructure across multiple Availability Zones
Ensuring application availability using Load Balancer and Auto Scaling
Securing and managing data using EBS and S3
Implementing backup and recovery using AWS Backup
**Architecture Diagram**

**Architecture Flow**
Internet → Application Load Balancer → EC2 Instances (Auto Scaling, Private Subnets) → 
EBS Storage → S3 (Static Content) → AWS Backup

**1. Networking Infrastructure (VPC)**
Configuration Details
Availability Zones: 2 AZs
Subnets:
2 Public Subnets (for Load Balancer)
2 Private Subnets (for Application Instances)
**Explanation**

This setup ensures fault tolerance by distributing resources across multiple AZs.
Public subnets handle incoming traffic, while private subnets securely host backend application servers.

**2. EC2 Deployment using Golden AMI**
Pre-configuration
Apache/Nginx installed
Application code deployed via User Data
**User Data Script**
#!/bin/bash

#Update and install dependencies
apt-get update -y
apt-get install -y nginx git

#Clean default web root
rm -rf /var/www/html/*

#Clone application repository
git clone https://github.com/schoolofdevops/html-sample-app /var/www/html/

#Set permissions
chown -R www-data:www-data /var/www/html/
chmod -R 755 /var/www/html/

#Restart service
systemctl restart nginx
Deployment Details
Launch EC2 instances using Golden AMI
Deploy instances in Public Subnets
**Explanation**

Golden AMI helps standardize infrastructure and reduces setup time during scaling.


**3. Storage Configuration (EBS)**
Configuration Details
Volume Type: gp3
Size: 20 GB
Encryption: Enabled (AWS KMS)
**Explanation**

EBS provides persistent block storage, and encryption ensures data security at rest.

**4. Application Load Balancer (ALB)**
ALB Configuration
Protocol: HTTP
Port: 80
Deployment: Across 2 AZs
Attributes
Cross-Zone Load Balancing: Enabled
Sticky Sessions: Enabled (1-hour duration)
**Explanation**

ALB distributes traffic evenly and ensures high availability and fault tolerance.

**5. Launch Template Setup**
Configuration Details
AMI: Golden AMI
Storage: 20 GB EBS (Encrypted)
**Explanation**

Launch Templates ensure consistent configuration during Auto Scaling.

**6. Auto Scaling Group (ASG)**
Scaling Parameters
Minimum Capacity: 2
Maximum Capacity: 6
Desired Capacity: 2

Scaling Policies
Scale Out: CPU > 60%
Scale In: CPU < 30%
Cooldown: 300 seconds

Termination Policy
OldestLaunchTemplate
**Explanation**

ASG automatically adjusts instances based on load, ensuring performance and cost efficiency.

**7. Storage (S3 & Backup)**

**S3 Configuration**
Used for static assets
Versioning: Enabled
Encryption: SSE-S3

**Backup Configuration**
EBS Snapshots: Daily
Retention: 7 Days (AWS Backup Vault)

**Explanation**
S3 provides scalable object storage
AWS Backup ensures data protection and recovery

**8. Validation**
Checks Performed
ALB health checks pass for all instances
Auto Scaling launches new instances under load
Static assets load correctly from S3
EBS snapshots are created successfully
**Explanation**

These checks confirm the system is fully functional and highly available.

**Final Outcome**

This architecture successfully implements:

High Availability across multiple AZs
Load-balanced traffic distribution
Auto Scaling based on demand
Secure and durable storage
Automated backup and recovery
