# Project 2: Store and Manage PDF Files Using Amazon S3

## Overview
This project builds a secure PDF storage workflow on AWS using Amazon S3 and IAM.

You will implement:
- PDF upload to a private S3 bucket
- Secure PDF download with time-limited pre-signed URLs
- Least-privilege IAM permissions for upload/download actions
- Basic hardening (encryption, public access blocking, logging)

## What Was Fixed in This README
- Replaced garbled text (`5â€“15`) with the correct range (`5-15`).
- Clarified that private S3 objects should be accessed via pre-signed URLs, not direct object URLs.
- Added concrete setup steps, validation rules, example IAM policies, and test checklist.
- Added an implementation section (CLI + Python examples) so this can be built and verified end-to-end.

## Learning Objectives
By the end of this project, you should be able to:
- Explain private vs public object access models in S3
- Create least-privilege IAM policies for object operations
- Upload and retrieve files securely using S3 APIs
- Generate and validate pre-signed URLs with expiration

## AWS Services Used

### 1) Amazon S3
Use S3 to:
- Create a bucket for PDFs
- Store objects with organized key prefixes
- Keep objects private by default
- Enforce encryption and access controls

### 2) IAM (Identity and Access Management)
Use IAM to:
- Create upload/download roles or users
- Scope permissions to only required bucket prefixes/actions
- Apply least-privilege and avoid root credentials

### 3) Optional Supporting Services
- **CloudTrail**: audit API activity
- **S3 Server Access Logs** or **CloudWatch** integration: track access patterns
- **KMS**: customer-managed encryption keys if stricter compliance is needed

## Suggested Architecture
`Client/App -> Auth Layer (IAM role or backend API) -> S3 Bucket (private) -> Pre-signed URL response`

Recommended pattern:
1. User requests upload/download from backend.
2. Backend authorizes user.
3. Backend generates pre-signed S3 URL.
4. User uploads/downloads directly with S3 using that temporary URL.

## Step-by-Step Build Plan

### Step 1: Create and Configure the Bucket
1. Create an S3 bucket in your preferred region.
2. Keep **Block Public Access** enabled (all options ON).
3. Enable bucket versioning (recommended).
4. Enable default encryption:
   - SSE-S3 (easy default), or
   - SSE-KMS (more control/audit)
5. Define key prefix structure, for example:
   - `lecture-notes/`
   - `resumes/`
   - `club-documents/`

### Step 2: Define IAM Permissions
Create separate policies for upload and download workflows.

#### Upload policy (example)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:PutObject"],
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/lecture-notes/*"
    },
    {
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME",
      "Condition": {
        "StringLike": {
          "s3:prefix": ["lecture-notes/*"]
        }
      }
    }
  ]
}
```

#### Download policy (example)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/lecture-notes/*"
    }
  ]
}
```

### Step 3: Implement Upload Rules
During upload:
- Accept only `.pdf` files
- Validate `Content-Type` as `application/pdf`
- Set size limits (example: max 10 MB)
- Use predictable keys: `lecture-notes/<user-id>/<timestamp>-<filename>.pdf`

### Step 4: Implement Secure Download
- Do not expose raw private object URLs.
- Generate pre-signed URLs with short expiry (recommended: 5-15 minutes).
- Return URLs only after authorization checks.

### Step 5: Test Public vs Private Access
- Confirm direct access to private objects fails without authorization.
- Confirm pre-signed URL works until expiry.
- Optionally test a controlled public-read object and document risks.

### Step 6: Security Hardening
- Keep bucket private and block public access.
- Rotate credentials; avoid long-lived access keys if possible.
- Prefer roles over IAM users for apps running on AWS compute.
- Enable logging/auditing.

## CLI Example (Quick Test)
```bash
aws s3 cp ./sample.pdf s3://YOUR_BUCKET_NAME/lecture-notes/sample.pdf
aws s3 presign s3://YOUR_BUCKET_NAME/lecture-notes/sample.pdf --expires-in 300
```

## Python Example (Pre-signed Download URL)
```python
import boto3

s3 = boto3.client("s3", region_name="us-east-1")

url = s3.generate_presigned_url(
    "get_object",
    Params={"Bucket": "YOUR_BUCKET_NAME", "Key": "lecture-notes/sample.pdf"},
    ExpiresIn=300,
)

print(url)
```

## Deliverables
- Architecture diagram
- IAM policy JSON files
- Working upload + secure download demo
- Security notes explaining design decisions

## Success Criteria
Project is complete when:
- PDF uploads succeed to private S3 paths
- Secure downloads work via time-limited pre-signed URLs
- IAM permissions are scoped to least privilege
- Security choices (private by default, encryption, logging) are documented

## Stretch Goals
- Add metadata indexing/search
- Build a small frontend uploader
- Add malware scanning before object acceptance
- Add lifecycle rules for archive/deletion
