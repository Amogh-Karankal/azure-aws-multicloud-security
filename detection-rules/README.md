# Detection Rules

Custom KQL detection rules for cross-cloud security monitoring in Microsoft Sentinel.

## Rules Overview

| File | Detection | MITRE ATT&CK | Severity |
|------|-----------|--------------|----------|
| 01-cross-cloud-brute-force.kql | Same IP attacking both clouds | T1110 | High |
| 02-aws-iam-user-created.kql | AWS IAM user creation | T1136 | Medium |
| 03-s3-bucket-public.kql | S3 bucket made public | T1530 | High |
| 04-cross-cloud-privilege-escalation.kql | Privilege escalation across clouds | T1078, T1098 | High |
| 05-impossible-travel-azure.kql | Multi-country logins | T1078 | Medium |

## How to Deploy

1. Go to **Microsoft Sentinel** → **Analytics**
2. Click **"+ Create"** → **"Scheduled query rule"**
3. Configure:
   - Name and description
   - Severity and MITRE tactics
   - Paste KQL query
   - Set schedule (e.g., every 5 minutes)
4. Click **"Create"**

## Rule Configuration

| Setting | Recommended Value |
|---------|-------------------|
| Query frequency | 5-15 minutes |
| Lookup period | 1-24 hours |
| Alert threshold | Greater than 0 |
| Event grouping | Alert per result |
