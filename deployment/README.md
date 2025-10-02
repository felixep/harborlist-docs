# Deployment Guide

## Prerequisites

### Required Tools
- **AWS CLI** v2.x - [Installation Guide](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
- **Node.js** v18+ - [Download](https://nodejs.org/)
- **AWS CDK** v2.x - `npm install -g aws-cdk`
- **Git** - Version control

## Infrastructure Deployment (Recommended)

### One-Command Deployment

```bash
# Navigate to infrastructure directory
cd infrastructure

# Install dependencies
npm install

# Build all components and deploy
./deploy.sh
cdk bootstrap  # First time only
cdk deploy
```

### Deployment Outputs
After successful deployment, note these outputs:
- `FrontendUrl`: Your application's public URL (CloudFront)
- `ApiUrl`: Backend API endpoint
- `MediaBucketName`: S3 bucket for media uploads
- `FrontendBucketName`: S3 bucket for frontend hosting

## Manual Deployment (Alternative)

### AWS Account Setup
1. **AWS Account** with appropriate permissions
2. **IAM User** with programmatic access
3. **AWS CLI** configured with credentials
4. **Domain Name** (optional, for custom domain)

### Required AWS Permissions
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "cloudformation:*",
        "iam:*",
        "lambda:*",
        "apigateway:*",
        "dynamodb:*",
        "s3:*",
        "cognito-idp:*",
        "opensearch:*",
        "ses:*",
        "cloudfront:*",
        "route53:*",
        "acm:*"
      ],
      "Resource": "*"
    }
  ]
}
```

## Environment Setup

### 1. Clone Repository
```bash
git clone https://github.com/your-org/boat-listing-platform.git
cd boat-listing-platform
```

### 2. Install Dependencies
```bash
# Install CDK dependencies
cd infrastructure
npm install

# Install frontend dependencies
cd ../frontend
npm install

# Install backend dependencies
cd ../backend
npm install
```

### 3. Configure Environment Variables
```bash
# Copy environment template
cp .env.example .env

# Edit configuration
nano .env
```

**Environment Variables:**
```bash
# AWS Configuration
AWS_REGION=us-east-1
AWS_ACCOUNT_ID=123456789012

# Application Configuration
APP_NAME=boat-listings
ENVIRONMENT=dev
DOMAIN_NAME=boatlistings.com

# Email Configuration
FROM_EMAIL=noreply@boatlistings.com
SUPPORT_EMAIL=support@boatlistings.com

# Feature Flags
ENABLE_VIDEO_UPLOAD=true
ENABLE_ANALYTICS=true
ENABLE_NOTIFICATIONS=true
```

## Deployment Steps

### Phase 1: Infrastructure Deployment

#### 1. Bootstrap CDK (First Time Only)
```bash
cd infrastructure
cdk bootstrap aws://123456789012/us-east-1
```

#### 2. Deploy Core Infrastructure
```bash
# Deploy in order
cdk deploy BoatListingsVpcStack
cdk deploy BoatListingsDataStack
cdk deploy BoatListingsAuthStack
cdk deploy BoatListingsApiStack
```

#### 3. Deploy Frontend Infrastructure
```bash
cdk deploy BoatListingsFrontendStack
```

### Phase 2: Application Deployment

#### 1. Deploy Backend Functions
```bash
cd ../backend
npm run build
npm run deploy
```

#### 2. Deploy Frontend Application
```bash
cd ../frontend
npm run build
amplify publish
```

### Phase 3: Configuration

#### 1. Configure Cognito User Pool
```bash
# Create admin user
aws cognito-idp admin-create-user \
  --user-pool-id us-east-1_XXXXXXXXX \
  --username admin \
  --user-attributes Name=email,Value=admin@boatlistings.com \
  --temporary-password TempPass123! \
  --message-action SUPPRESS
```

#### 2. Configure SES Email
```bash
# Verify sender email
aws ses verify-email-identity \
  --email-address noreply@boatlistings.com

# Create email templates
aws ses create-template \
  --template file://email-templates/contact-notification.json
```

#### 3. Configure Domain (Optional)
```bash
# Request SSL certificate
aws acm request-certificate \
  --domain-name boatlistings.com \
  --subject-alternative-names "*.boatlistings.com" \
  --validation-method DNS

# Update CloudFront distribution
cdk deploy BoatListingsFrontendStack --parameters DomainName=boatlistings.com
```

## Environment-Specific Deployments

### Development Environment
```bash
export ENVIRONMENT=dev
cdk deploy --all --parameters Environment=dev
```

### Staging Environment
```bash
export ENVIRONMENT=staging
cdk deploy --all --parameters Environment=staging
```

### Production Environment
```bash
export ENVIRONMENT=prod
cdk deploy --all --parameters Environment=prod
```

## Post-Deployment Configuration

### 1. Verify Deployment
```bash
# Check API Gateway endpoints
curl https://api-dev.boatlistings.com/health

# Check frontend
curl https://dev.boatlistings.com

# Check authentication
aws cognito-idp list-users --user-pool-id us-east-1_XXXXXXXXX
```

### 2. Load Sample Data (Development Only)
```bash
cd scripts
node load-sample-data.js --environment dev
```

### 3. Configure Monitoring
```bash
# Create CloudWatch dashboard
aws cloudwatch put-dashboard \
  --dashboard-name "BoatListings-${ENVIRONMENT}" \
  --dashboard-body file://monitoring/dashboard.json

# Set up alarms
aws cloudwatch put-metric-alarm \
  --alarm-name "HighErrorRate-${ENVIRONMENT}" \
  --alarm-description "High error rate detected" \
  --metric-name ErrorRate \
  --namespace AWS/ApiGateway \
  --statistic Average \
  --period 300 \
  --threshold 5.0 \
  --comparison-operator GreaterThanThreshold
```

## Rollback Procedures

### Application Rollback
```bash
# Rollback frontend
amplify publish --branch previous-version

# Rollback backend functions
cd backend
npm run rollback --version previous
```

### Infrastructure Rollback
```bash
# Rollback to previous CDK version
cdk deploy --rollback
```

### Database Rollback
```bash
# Restore DynamoDB from backup
aws dynamodb restore-table-from-backup \
  --target-table-name boat-listings-dev \
  --backup-arn arn:aws:dynamodb:us-east-1:123456789012:table/boat-listings-dev/backup/01234567890123-abcdefgh
```

## Troubleshooting

### Common Issues

#### 1. CDK Bootstrap Issues
```bash
# Error: "Need to perform AWS CDK bootstrap"
cdk bootstrap aws://ACCOUNT-ID/REGION

# Error: "Version mismatch"
npm update -g aws-cdk
```

#### 2. Permission Issues
```bash
# Error: "Access Denied"
# Check IAM permissions and policies
aws sts get-caller-identity
aws iam get-user
```

#### 3. Domain Configuration Issues
```bash
# Error: "Certificate not found"
# Verify certificate in us-east-1 region
aws acm list-certificates --region us-east-1

# Error: "DNS validation failed"
# Check Route53 hosted zone
aws route53 list-hosted-zones
```

#### 4. API Gateway Issues
```bash
# Error: "Internal Server Error"
# Check Lambda function logs
aws logs describe-log-groups --log-group-name-prefix /aws/lambda/boat-listings

# Get recent logs
aws logs filter-log-events \
  --log-group-name /aws/lambda/boat-listings-api \
  --start-time $(date -d '1 hour ago' +%s)000
```

### Health Check Commands
```bash
# Check all services
./scripts/health-check.sh

# Check specific service
curl -f https://api.boatlistings.com/health || echo "API is down"
```

## Monitoring and Maintenance

### Daily Checks
- [ ] API Gateway health status
- [ ] Lambda function error rates
- [ ] DynamoDB throttling metrics
- [ ] S3 storage costs
- [ ] CloudFront cache hit rates

### Weekly Maintenance
- [ ] Review CloudWatch logs for errors
- [ ] Check DynamoDB backup status
- [ ] Monitor storage costs and usage
- [ ] Review security group rules
- [ ] Update dependencies

### Monthly Tasks
- [ ] Review and optimize costs
- [ ] Update CDK and dependencies
- [ ] Security audit and updates
- [ ] Performance optimization review
- [ ] Backup and disaster recovery testing

## Cost Optimization

### Automated Cost Controls
```bash
# Set up billing alerts
aws budgets create-budget \
  --account-id 123456789012 \
  --budget file://budgets/monthly-budget.json

# Configure lifecycle policies
aws s3api put-bucket-lifecycle-configuration \
  --bucket boat-listings-media-dev \
  --lifecycle-configuration file://s3-lifecycle.json
```

### Manual Optimizations
- Review DynamoDB read/write capacity
- Optimize Lambda memory allocation
- Configure S3 storage classes
- Review CloudFront caching rules
- Monitor OpenSearch cluster size

## Security Checklist

### Pre-Production
- [ ] Enable AWS Config rules
- [ ] Configure AWS GuardDuty
- [ ] Set up AWS Security Hub
- [ ] Enable VPC Flow Logs
- [ ] Configure AWS WAF rules
- [ ] Enable CloudTrail logging
- [ ] Review IAM policies (least privilege)
- [ ] Enable MFA for admin accounts

### Post-Production
- [ ] Regular security scans
- [ ] Monitor suspicious activities
- [ ] Update security patches
- [ ] Review access logs
- [ ] Audit user permissions
- [ ] Test incident response procedures
