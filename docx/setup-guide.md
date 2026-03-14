# Setup Guide: Azure-AWS Multi-Cloud Security

Complete implementation guide for integrating Azure and AWS security.

## Prerequisites

| Requirement | Details |
|-------------|---------|
| Azure Subscription | With Microsoft Sentinel enabled |
| AWS Account | Free tier eligible |
| Permissions | Global Admin (Azure), IAM Admin (AWS) |

## Phase 1: AWS Account Setup (30 min)

### 1.1 Create AWS Account
1. Go to https://aws.amazon.com/free/
2. Create account with MFA enabled
3. Create IAM admin user (don't use root)

### 1.2 Enable MFA
1. IAM → Security credentials → Assign MFA device
2. Configure for both root and admin user

## Phase 2: Identity Federation (2 hours)

### 2.1 Enable IAM Identity Center
1. AWS Console → IAM Identity Center
2. Click "Enable"
3. Note the **AWS access portal URL**

### 2.2 Create Azure Enterprise App
1. Azure Portal → Microsoft Entra ID → Enterprise applications
2. Create your own application: `AWS IAM Identity Center`
3. Configure Single sign-on → SAML

### 2.3 Configure SAML
1. Get from AWS:
   - IAM Identity Center Issuer URL
   - IAM Identity Center ACS URL
2. Enter in Azure SAML configuration:
   - Identifier (Entity ID) = Issuer URL
   - Reply URL = ACS URL

### 2.4 Download Federation Metadata
1. Azure → SAML Certificates → Download Federation Metadata XML
2. Upload to AWS IAM Identity Center

### 2.5 Enable SCIM Provisioning
1. AWS → IAM Identity Center → Settings → Automatic provisioning → Enable
2. Copy SCIM endpoint and access token
3. Azure → Enterprise app → Provisioning → Configure with SCIM values
4. Start provisioning

### 2.6 Assign Users and Permission Sets
1. Assign users to Enterprise app in Azure
2. Create Permission Set in AWS (e.g., AdministratorAccess)
3. Assign synced users to AWS accounts

### 2.7 Test SSO
1. Go to AWS access portal URL
2. Login with Azure credentials
3. Access AWS Console via SSO

## Phase 3: CloudTrail Configuration (1 hour)

### 3.1 Create CloudTrail Trail
1. AWS → CloudTrail → Create trail
2. Name: `security-audit-trail`
3. Create new S3 bucket
4. Enable log file validation

### 3.2 Create SQS Queue
1. AWS → SQS → Create queue
2. Name: `sentinel-cloudtrail-queue`
3. Configure access policy for S3

### 3.3 Configure S3 Event Notifications
1. S3 bucket → Properties → Event notifications
2. Create notification for `.json.gz` files
3. Send to SQS queue

### 3.4 Create IAM Role for Sentinel
1. Create OIDC Identity Provider:
   - URL: `https://sts.windows.net/YOUR-TENANT-ID/`
   - Audience: `api://1462b192-27f7-4cb9-8523-0f4ecb54b47e`
2. Create IAM Role: `MicrosoftSentinelRole`
3. Attach policies:
   - AmazonS3ReadOnlyAccess
   - AmazonSQSReadOnlyAccess
   - AWSCloudTrailReadOnlyAccess
4. Configure trust policy with Microsoft account ID and Workspace ID

## Phase 4: GuardDuty (30 min)

1. AWS → GuardDuty → Enable
2. Generate sample findings for testing

## Phase 5: Connect AWS to Sentinel (1 hour)

### 5.1 Install Connector
1. Sentinel → Content hub → Search "Amazon Web Services"
2. Install the solution

### 5.2 Configure Connection
1. Sentinel → Data connectors → Amazon Web Services S3
2. Add connection:
   - Role ARN: Your MicrosoftSentinelRole ARN
3. Wait for data (15-30 minutes)

### 5.3 Verify Data
```kql
AWSCloudTrail
| take 20
```

## Phase 6: Defender for Cloud (1 hour)

### 6.1 Add AWS Environment
1. Defender for Cloud → Environment settings → Add environment → AWS
2. Configure connector basics

### 6.2 Select Plans
- Enable: Foundational CSPM (Free)
- Disable: All paid plans

### 6.3 Deploy CloudFormation
1. Download CloudFormation template from Azure
2. AWS → CloudFormation → Create stack
3. Upload template, wait for CREATE_COMPLETE

## Phase 7: Detection Rules (1 hour)

Deploy the 5 KQL detection rules from `/detection-rules/` folder:

1. **Cross-Cloud Brute Force** (T1110)
2. **AWS IAM User Created** (T1136)
3. **S3 Bucket Made Public** (T1530)
4. **Cross-Cloud Privilege Escalation** (T1078, T1098)
5. **Impossible Travel Azure** (T1078)

## Phase 8: Dashboard (30 min)

1. Sentinel → Workbooks → Add workbook
2. Add query panels for:
   - Events by cloud (pie chart)
   - Failed logins over time
   - Top AWS events
   - AWS activity by user

## Phase 9: Testing (30 min)

### Simulate Cross-Cloud Brute Force
1. Try wrong password at portal.azure.com (3-5 times)
2. Try wrong password at AWS console (same IP)

### Simulate IAM User Creation
1. AWS → IAM → Users → Create user

### Simulate Impossible Travel
1. Login to Azure normally
2. Connect to VPN (different country)
3. Login to Azure again

## Troubleshooting

### SSO Not Working
- Verify SAML URLs match exactly
- Check SCIM sync status (up to 40 min)
- Verify user is assigned in Azure

### No AWS Data in Sentinel
- Verify IAM role trust policy
- Check SQS permissions
- Wait 30+ minutes for initial sync

### Defender Not Showing AWS
- Verify CloudFormation completed
- Can take up to 24 hours

## Cost Estimate

| Service | Cost |
|---------|------|
| AWS (IAM, CloudTrail, GuardDuty trial) | Free |
| Azure Sentinel | Existing subscription |
| Defender CSPM Foundational | Free |
| **Total** | **$0-5/month** |
