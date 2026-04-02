# Enterprise Disaster Recovery & Backup Architecture

## Project Overview

This project demonstrates enterprise-grade AWS storage and backup strategies including:

- AWS Backup with Vault & lifecycle policies
- Advanced EBS features (Multi-Attach + live resize)
- EFS shared storage with lifecycle management
- Hybrid Storage using File Gateway (simulated on EC2)

It also includes disaster recovery validation scenarios aligned with real-world RTO/RPO goals.

---

## Architecture Diagram

**Components:**

- EC2 Instances (Multi-AZ)
- EBS io2 Multi-Attach Volume
- EFS File System
- AWS Backup Vault (KMS Encrypted)
- S3 Bucket (Storage Gateway Backend)
- File Gateway (on EC2)
- Cross-Region Backup (DR Region)

---

## Services Used

- AWS Backup
- Amazon EBS
- Amazon EFS
- AWS KMS
- AWS Storage Gateway
- Amazon S3
- Amazon EC2

---

# TASK 1 — AWS Backup Configuration

## Objective

Configure centralized backup with encryption and lifecycle policies.

## Steps

### 1. Create Backup Vault

- Go to **AWS Backup → Backup Vaults**
- Create vault:
  - **Name:** `charan-prod-backup-vault`
  - **Encryption:** Use KMS Key

### 2. Create Backup Plan

- Go to **Backup Plans → Create Plan**

### Plan Configuration

- **Hourly Backup (EBS)**
  - Frequency: Every 1 hour
  - Retention: 24 hours

- **Daily Backup (EFS + AMI)**
  - Frequency: Daily
  - Retention: 30 days

- **Monthly Cross-Region Copy**
  - Destination Region: `eu-west-1`
  - Retention: 1 year

### 3. Assign Resources

Assign:

- EBS volumes
- EFS file system
- EC2 instances (for AMI backups)

## Validation

- Trigger manual backup
- Verify job success

---

# TASK 2 — EBS Advanced Features

## Objective

Demonstrate high-performance storage with zero downtime scaling.

## Steps

### 1. Create io2 Volume

- Size: **100 GB**
- Type: **io2**
- Enable: **Multi-Attach**

### 2. Attach to 2 EC2 Instances

- Attach same volume to both instances

### 3. Mount Volume

```bash
lsblk
sudo mkfs -t ext4 /dev/nvme1n1
sudo mount /dev/nvme1n1 /data
```

### 4. Resize Volume (100GB → 150GB)

#### Step 1: Modify Volume

- Increase size to **150GB** in console

#### Step 2: Extend Filesystem

```bash
sudo growpart /dev/nvme1n1
sudo resize2fs /dev/nvme1n1
```

## Validation

- Verify both instances can read/write
- Confirm new disk size

```bash
df -h
```

---

# TASK 3 — EFS Setup

## Objective

Set up shared file storage across multiple AZs with lifecycle policies.

## Steps

### 1. Create EFS File System

- Enable across **2 AZs**
- Performance mode: **General Purpose**

### 2. Configure Lifecycle Policy

- IA transition: **30 days**
- Archive transition: **90 days**

### 3. Mount on EC2 Instances (3 Instances)

```bash
sudo apt install nfs-common -y
sudo mount -t nfs4 <efs-endpoint>:/ /mnt/efs
```

## Validation

Write file from one instance:

```bash
echo "test" > /mnt/efs/file.txt
```

Fix permission issue:

```bash
sudo chown ubuntu:ubuntu /home/ubuntu/efs
```

Verify visibility on other instances.

---

# Storage Gateway (Hybrid Setup)

## Objective

Simulate on-prem to cloud file sync using File Gateway.

## Steps

### 1. Launch EC2 for File Gateway

- Instance Type: **m5.large** (recommended)
- Root volume: **80GB**
- Minimum usable: **30GB**
- Attach additional disk: **150GB** for caching

### 2. Install File Gateway

- Deploy AWS Storage Gateway
- Activate gateway

### 3. Configure File Share (NFS)

- Backend: **S3 Bucket**
- Enable: **Cross-region replication for DR**

### 4. Mount NFS Share

```bash
sudo mount -t nfs <gateway-ip>:/share /mnt/gateway
```

### 5. Write Data

```bash
echo "Hello Hybrid Cloud" > /mnt/gateway/test.txt
```

## Validation

- Verify file in S3 bucket
- Confirm replication in DR region

---

# Disaster Recovery Testing

## Scenario 1 — EFS File Deletion

- Delete file from EFS
- Restore using AWS Backup

## Scenario 2 — Backup Restore

- Restore EBS / EFS from backup
- Verify data integrity

---

# Key Concepts Covered

- AWS Backup (Vaults, Plans, Lifecycle)
- KMS Encryption
- EBS Multi-Attach
- Live Volume Resize
- EFS Lifecycle Management
- Hybrid Storage (File Gateway)
- Disaster Recovery (RTO/RPO)

---

# Validation Checklist

- [x] Manual backup triggered and completed
- [x] EFS restore successful after deletion
- [x] Multi-Attach works across 2 instances
- [x] EBS resized without downtime
- [x] Storage Gateway syncs files to S3
- [x] Cross-region replication verified
