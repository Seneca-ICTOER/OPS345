---
id: lab7
title: Lab 7 - Simple Storage Service (S3) Glacier
sidebar_position: 7
description: Configure backups using S3 Glacier
---

# Lab 7 - Simple Storage Service (S3) Glacier

# Under Development

## Overview

## Create an S3 bucket
- S3
- Click **Create bucket**
- On the **Create bucket** page
  - Bucket name: **senecausername-wordpressbackup**
  - Use the rest of the defaults
  - Click **Create bucket**

## Create a database snapshot
- Aurora and RDS > Snapshots
- Click **Take snapshot**
- On the **Take DB Snapshot** page select
  - Snapshot type: **DB instance**
  - DB instance: **wordpress-db**
  - Snapshot name: **initialbackup**
  - Click **Take snapshot**

## Creating an IAM Role
- IAM
  - Click on **Roles**
  - Click **Create role**
  - Use case select: **S3**
  - Click **Next**

  - Add permissions. Search for
    - **AmazonS3FullAccess**
    - **AmazonRDSDataFullAccess**
  - Click **Next**

  - Name, review and create
    - Role name: rds-s3-backup
    - Click **Create role**

## Creating an AWS Key Management Service (KMS) key
- Key Management Service (KMS)
  - Click **Create a key**

- Under **Configure key**
  - Key type: **Asymmetric**
  - Key usage: **Encrypt and decrypt**
  - Key spec: **RSA_4096**
  - Click **Next**

- Under **Add labels**
  - Alias: **wpbackupkey**
  - Click **Next**

- Under **Define key administrative permissions - optional**
  - Accept the defaults and click **Next**

- Under **Define key usage permissions - optional**
  - (use the search or scroll)
    - Select **LabRole** 
    - Select **AWSServiceRoleForRDS**

- Under **Edit key policy - optional**
  - Accept the defaults and click **Next**

- Verify everything looks ok. Scroll to the bottom and click **Finish**

- Click on your **Key ID** and copy the **ARN**. You will need this later.

## Exporting your backup to an S3 bucket
- Aurora and RDS > Snapshots
- Check the box beside **initialbackup**
- In the actions dropdown select **Export to Amazon S3**

- Export to Amazon S3 page:
  - Settings > Export identifier **wpbackup-month-date-year** ie: **wpbackup-jul-5-2025**
  - Make sure the S3 destination is set to your S3 bucket: **senecausername-wordpressbackup**
  - IAM role: **LabRole**
