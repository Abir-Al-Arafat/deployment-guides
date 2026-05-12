# General Backend Infrastructure: MongoDB & S3 Backup Configuration Guide

This document outlines the architecture and configuration for implementing a local MongoDB database with an automated backup system utilizing Amazon S3 for general backend projects.

## 1. The Necessity of Local Hosting and Automated Backups

When managing a production backend, the choice of database infrastructure significantly impacts application performance and reliability. Transitioning from a managed cloud service to a self-hosted environment is often driven by several key factors:

- **Performance Optimization**: Hosting the database on the same infrastructure (or within the same VPC) as the backend application minimizes network latency. This results in faster query execution and improved overall application responsiveness.
- **Cost Efficiency**: For growing applications, self-hosting can be more cost-effective than managed services, providing full control over resource allocation.
- **Data Sovereignty**: Local hosting allows for strict control over where data resides, which is essential for meeting specific regulatory or compliance requirements.
- **Disaster Recovery Requirement**: Unlike managed services that provide automated backups, self-hosted databases require a custom backup strategy. An off-site backup (e.g., to Amazon S3) is critical to protect against hardware failure, file system corruption, or accidental data loss.

## 2. Amazon S3 Configuration

An S3 bucket acts as a durable, off-site storage solution for both sensitive database backups and application-specific media.

### Recommended Bucket Settings

- **Region-Specific Placement**: The bucket should be created in the same AWS region as the server to ensure high-speed data transfers.
- **Namespace Management**: Standard buckets use a global namespace requiring unique naming.
- **Object Ownership**: Disabling ACLs and enforcing "Bucket owner preferred" is the modern security standard for centralized control.
- **Encryption at Rest**: Utilizing S3-managed keys (SSE-S3) provides seamless, automated encryption for all stored objects.

### Logical Directory Structure

Data should be organized using prefixes (folders) to separate different data types:

- `db-backups/`: Dedicated to compressed database archives. This directory must remain strictly private.
- `media/`: Used for application assets, user uploads, or static content.

### Security and Access Control

A granular **Bucket Policy** should be implemented to allow public read access _only_ to specific media paths while keeping system backups protected:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadForMediaOnly",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<your-bucket-name>/media/*"
    }
  ]
}
```

## 3. Server & IAM Integration

Security is best managed through **IAM Roles** attached to the compute resource (e.g., EC2), eliminating the need for hardcoded credentials.

### Required Permissions

The following policy should be attached to the server's IAM role to allow it to interact with the specific S3 bucket:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:PutObject", "s3:GetObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::<your-bucket-name>",
        "arn:aws:s3:::<your-bucket-name>/*"
      ]
    }
  ]
}
```

## 4. Automated Backup Implementation

The backup system relies on a standardized automation script and a scheduling daemon (like Cron).

### Standardized Backup Script (`backup_db.sh`)

This script automates the process of dumping the database, compressing the file, and transferring it to S3.

```bash
#!/bin/bash

# Configuration
DB_NAME="<your_db_name>"
BACKUP_NAME="${DB_NAME}_$(date +%Y%m%d_%H%M%S).gz"
DEST_DIR="/home/ubuntu/backups"
S3_BUCKET="s3://<your-bucket-name>/db-backups/"
MONGODB_URI="mongodb://localhost:27017/${DB_NAME}?replicaSet=rs0"

# Ensure local directory exists
mkdir -p $DEST_DIR

# 1. Generate compressed backup
mongodump --uri=\"$MONGODB_URI\" --archive=\"$DEST_DIR/$BACKUP_NAME\" --gzip

# 2. Synchronize to S3
/usr/local/bin/aws s3 cp \"$DEST_DIR/$BACKUP_NAME\" \"$S3_BUCKET\"

# 3. Clean up temporary local storage
rm \"$DEST_DIR/$BACKUP_NAME\"
```

### Automation via Cron

To ensure consistent data protection, the script should be scheduled to run during off-peak hours.

- **Cron Example**: `0 2 * * * /bin/bash /path/to/backup_db.sh >> /path/to/backup_log.txt 2>&1`
