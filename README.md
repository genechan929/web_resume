# genechan.ca Resume Site Architecture

This repository contains a static resume website hosted on AWS with automated deployment via GitHub Actions.

## Architecture Overview

This setup leverages AWS services for hosting and content delivery, with GitHub Actions providing continuous deployment capabilities.

### Architecture Diagram

```mermaid
graph TB
    Dev[Developer] -->|git push| GH[GitHub Repository]
    GH -->|triggers| GA[GitHub Actions]
    GA -->|authenticate| AWS[AWS Account]
    GA -->|sync files| S3[S3 Bucket]
    GA -->|invalidate cache| CF[CloudFront Distribution]
    S3 -->|serves content| CF
    CF -->|delivers content| Users[End Users (genechan.ca)]
    DNS[Route 53 / DNS Provider] -->|points to| CF
    
    subgraph "AWS Infrastructure"
        S3
        CF
        AWS
    end
    
    subgraph "CI/CD Pipeline"
        GH
        GA
    end
    
    style S3 fill:#ff9900
    style CF fill:#ff9900
    style GA fill:#2088ff
    style GH fill:#333
```

## Components

### 1. GitHub Repository
- **Purpose**: Source code management and version control
- **Trigger**: Push events to main branch initiate deployment
- **Secrets**: Stores AWS credentials and configuration

### 2. GitHub Actions
- **Purpose**: Automated CI/CD pipeline
- **Workflow**: `.github/workflows/deploy.yml`
- **Actions**:
  - Checkout code
  - Configure AWS credentials
  - Sync files to S3
  - Invalidate CloudFront cache

### 3. AWS S3 (Simple Storage Service)
- **Bucket Name**: `S3_BUCKET`
- **Region**: `S3_BUCKET_REGION`
- **Purpose**: Static resume website hosting
- **Configuration**:
  - Website hosting enabled
  - Public read access for web content
  - Bucket policy allowing CloudFront access

### 4. AWS CloudFront
- **Purpose**: Content Delivery Network (CDN)
- **Benefits**:
  - Global content caching
  - HTTPS/SSL termination
  - Custom domain support
  - Edge locations for faster content delivery

### 5. DNS Configuration
- **Purpose**: Domain name resolution for genechan.ca
- **Setup**: DNS A/CNAME record pointing to CloudFront distribution

## Deployment Flow

1. **Code Changes**: Developer pushes changes to GitHub repository
2. **Trigger**: GitHub Actions workflow triggered on push to main branch
3. **Authentication**: GitHub Actions authenticates with AWS using stored secrets
4. **Sync**: Files are synchronized to S3 bucket (with cleanup of deleted files)
5. **Cache Invalidation**: CloudFront cache is invalidated to serve fresh content
6. **Live**: Updated content is available globally via CloudFront

## Configuration

### GitHub Secrets
The following secrets must be configured in GitHub repository settings:

| Secret Name | Description |
|-------------|-------------|
| `AWS_ACCESS_KEY_ID` | AWS IAM user access key |
| `AWS_SECRET_ACCESS_KEY` | AWS IAM user secret key |
| `S3_BUCKET` | S3 bucket name |
| `CLOUDFRONT_DISTRIBUTION_ID` | CloudFront distribution ID |

### AWS IAM Permissions
The AWS user requires the following permissions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::S3_BUCKET",
        "arn:aws:s3:::S3_BUCKET/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "cloudfront:CreateInvalidation"
      ],
      "Resource": "*"
    }
  ]
}
```


## Troubleshooting

### Common Issues

1. **Deployment Fails**
   - Check AWS credentials are correct
   - Verify IAM permissions
   - Review GitHub Actions logs

2. **Content Not Updating**
   - Ensure CloudFront invalidation completed
   - Check browser cache (force refresh)
   - Verify S3 sync completed successfully

3. **403 Errors**
   - Check S3 bucket policy
   - Verify CloudFront origin access control
   - Ensure files have correct permissions


## Development Workflow

1. Clone repository locally
2. Make changes to website files
3. Test changes locally
4. Commit and push to main branch
5. GitHub Actions automatically deploys changes
6. Verify changes are live on the website

---

*Last updated: 2025-07-05*