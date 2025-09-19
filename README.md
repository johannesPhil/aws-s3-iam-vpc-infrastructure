# CloudLaunch - AWS Cloud Infrastructure Assignment

## Project Overview

CloudLaunch is a lightweight platform demonstration showcasing AWS core services including S3, IAM, and VPC. This project implements secure static website hosting with controlled access permissions and a properly designed network infrastructure.

## Architecture Summary

- **Static Website**: Hosted on S3 with public read access
- **Private Document Storage**: S3 bucket with restricted IAM access
- **Network Infrastructure**: 3-tier VPC with proper isolation
- **Security**: Least-privilege IAM policies and network segmentation

---

## Task 1: S3 Static Website Hosting + IAM

### S3 Buckets Created

#### 1. Website Bucket (`johannes-cloudlaunch-site-bucket`)

- **Purpose**: Hosts the public CloudLaunch website
- **Configuration**: Static website hosting enabled
- **Access**: Publicly readable (anonymous users)
- **Website URL**: `http://johannes-cloudlaunch-site-bucket.s3-website.eu-north-1.amazonaws.com/`

#### 2. Private Bucket (`johannes-cloudlaunch-private-bucket`)

- **Purpose**: Secure document storage
- **Access**: IAM user can GetObject and PutObject only
- **Security**: No public access, no delete permissions

#### 3. Visible Only Bucket (`johannes-cloudlaunch-visible-only-bucket`)

- **Purpose**: Demonstrates restricted visibility permissions
- **Access**: IAM user can list/see bucket but cannot access contents
- **Security**: Explicit deny policy for object operations

### IAM User Configuration

**Username**: `cloudlaunch-user`
**Access Type**: AWS Management Console access
**Password Policy**: Must change password on first login

### IAM Policy (CloudLaunchUserPolicy)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3ListAllBuckets",
      "Effect": "Allow",
      "Action": "s3:ListAllMyBuckets",
      "Resource": "*"
    },
    {
      "Sid": "S3BucketAccess",
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::johannes-cloudlaunch-site-bucket",
        "arn:aws:s3:::johannes-cloudlaunch-private-bucket",
        "arn:aws:s3:::johannes-cloudlaunch-visible-only-bucket"
      ]
    },
    {
      "Sid": "S3PrivateBucketAccess",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": ["arn:aws:s3:::johannes-cloudlaunch-private-bucket/*"]
    },
    {
      "Sid": "S3SiteBucket",
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": ["arn:aws:s3:::johannes-cloudlaunch-site-bucket/*"]
    },
    {
      "Sid": "DenyVisibleOnlyBucketContents",
      "Effect": "Deny",
      "Action": ["s3:GetObject", "s3:PutObject", "s3:DeleteObject"],
      "Resource": ["arn:aws:s3:::johannes-cloudlaunch-visible-only-bucket/*"]
    },
    {
      "Sid": "VPCReadOnlyAccess",
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeVpcs",
        "ec2:DescribeSubnets",
        "ec2:DescribeRouteTables",
        "ec2:DescribeSecurityGroups",
        "ec2:DescribeInternetGateways"
      ],
      "Resource": "*"
    }
  ]
}
```

---

## Task 2: VPC Network Design

### VPC Architecture

**Name**: `cloudlaunch-vpc`
**CIDR Block**: `10.0.0.0/16`

### Subnets

#### Public Subnet

- **Name**: `cloudlaunch-public-subnet`
- **CIDR**: `10.0.1.0/24`
- **Purpose**: Load balancers and public-facing services
- **Internet Access**: Yes (via Internet Gateway)

#### Application Subnet

- **Name**: `cloudlaunch-app-subnet`
- **CIDR**: `10.0.2.0/24`
- **Purpose**: Application servers and business logic
- **Internet Access**: No (private subnet)

#### Database Subnet

- **Name**: `cloudlaunch-db-subnet`
- **CIDR**: `10.0.3.0/28`
- **Purpose**: Database services and sensitive data
- **Internet Access**: No (private subnet)

### Network Components

#### Internet Gateway

- **Name**: `cloudlaunch-igw`
- **Purpose**: Provides internet connectivity to public subnet

#### Route Tables

1. **Public Route Table** (`cloudlaunch-public-rt`)

   - Route: `0.0.0.0/0` → Internet Gateway
   - Associated with: Public subnet

2. **App Route Table** (`cloudlaunch-app-rt`)

   - No internet route (private)
   - Associated with: Application subnet

3. **Database Route Table** (`cloudlaunch-db-rt`)
   - No internet route (private)
   - Associated with: Database subnet

#### Security Groups

1. **App Security Group** (`cloudlaunch-app-sg`)

   - Inbound: HTTP (port 80) from VPC CIDR (10.0.0.0/16)
   - Purpose: Control access to application servers

2. **Database Security Group** (`cloudlaunch-db-sg`)
   - Inbound: MySQL (port 3306) from app subnet only (10.0.2.0/24)
   - Purpose: Restrict database access to app tier only

---

## Access Information

### Console Sign-in Details

- **Console URL**: `https://491085392395.signin.aws.amazon.com/console`
- **Username**: `cloudlaunch-user`
- **Initial Password**: [Cloud-Launch]

### Testing the Setup

#### S3 Permissions Verification:

- ✅ User can see all three buckets
- ✅ User can read files from site bucket
- ✅ User can upload/download from private bucket
- ✅ User can see visible-only bucket but gets "Access Denied" when accessing contents
- ✅ User cannot delete objects from any bucket

#### VPC Permissions Verification:

- ✅ User can view VPC and all components
- ✅ User cannot create, modify, or delete VPC resources

---

## Network Architecture Diagram

```
Internet
    |
    ▼
[Internet Gateway]
    |
    ▼
┌─────────────────────────────────────────────────────────┐
│                VPC (10.0.0.0/16)                        │
│                                                         │
│  ┌─────────────────┐    ┌─────────────────┐             │
│  │  Public Subnet  │    │  App Subnet     │             │
│  │  10.0.1.0/24    │    │  10.0.2.0/24    │             │
│  │                 │    │  (Private)      │             │
│  │ [Load Balancer] │◄──►│ [App Servers]   │             │
│  └─────────────────┘    └─────────────────┘             │
│                                   │                     │
│                                   ▼                     │
│                         ┌─────────────────┐             │
│                         │  DB Subnet      │             │
│                         │  10.0.3.0/28    │             │
│                         │  (Private)      │             │
│                         │ [Database]      │             │
│                         └─────────────────┘             │
└─────────────────────────────────────────────────────────┘
```

---

## Security Features

### Network Security

- **3-tier architecture** with proper isolation
- **Private subnets** have no internet access
- **Security groups** implement least-privilege access
- **Route table isolation** prevents unauthorized traffic flow

### IAM Security

- **Least-privilege principle** applied to all permissions
- **Explicit deny policies** for sensitive operations
- **Resource-specific ARNs** limit access scope
- **No delete permissions** anywhere in the policy

### Best Practices Implemented

- Unique bucket naming convention
- Forced password change on first login
- Proper CIDR block planning
- Separated route tables for different subnet tiers

---

## Cost Optimization

All resources created are within **AWS Free Tier** limits:

- S3 buckets: Free to create
- S3 storage: Well under 5GB limit
- VPC components: No additional charges
- IAM users and policies: Always free
- Static website hosting: No additional S3 charges

**Total Monthly Cost**: $0 (within free tier usage)

---

## Future Enhancements

### Potential Improvements:

1. **CloudFront Distribution**: Add CDN for global performance
2. **SSL Certificate**: Implement HTTPS with AWS Certificate Manager
3. **Auto Scaling**: Add EC2 auto-scaling groups in app subnet
4. **RDS Database**: Deploy managed database in database subnet
5. **Monitoring**: Implement CloudWatch logging and alerts
6. **CI/CD Pipeline**: Add automated deployment with CodePipeline

### Production Considerations:

- Multi-AZ deployment for high availability
- NAT Gateway for private subnet internet access
- Database encryption at rest and in transit
- VPC Flow Logs for network monitoring
- AWS Config for compliance monitoring

---

## Conclusion

This CloudLaunch implementation demonstrates a solid understanding of AWS core services and security best practices. The infrastructure provides a scalable foundation that can support real-world applications while maintaining strict security controls and cost optimization.

The combination of properly configured S3 buckets, fine-grained IAM policies, and well-designed VPC architecture showcases enterprise-grade cloud infrastructure design suitable for production workloads.
