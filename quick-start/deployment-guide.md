# Deployment Guide

## 🚀 Overview

This guide covers deploying the boat listing platform to AWS using CDK across multiple environments. The platform is designed to work with custom domains by default for production deployments.

**Current Status**: ✅ **DEPLOYED & OPERATIONAL**

## 🌐 Custom Domains Architecture

### Why Custom Domains Are Required

1. **🔒 Security**: Cloudflare provides DDoS protection, SSL termination, and security features
2. **🚀 Performance**: Global CDN with edge caching for better user experience
3. **🛡️ Origin Protection**: API Gateway and S3 are protected behind Cloudflare
4. **📊 Analytics**: Cloudflare provides detailed traffic analytics and monitoring
5. **🔧 Flexibility**: Easy SSL management, custom headers, and routing rules

### When to Use Custom Domains

✅ **Production deployments**
✅ **Staging environments** 
✅ **Demo environments**
✅ **Any public-facing deployment**

### When You Can Skip Custom Domains

❌ **Local development only** (use `useCustomDomains=false`)
❌ **Internal testing** (temporary setups)
❌ **CI/CD pipeline testing** (automated tests)

⚠️ **Warning**: Without custom domains, you'll lose security features, performance benefits, and the frontend may not work correctly due to CORS and routing configurations.

## 📋 Prerequisites

### AWS Account Setup
- AWS account with administrative access
- AWS CLI installed and configured
- AWS CDK CLI installed: `npm install -g aws-cdk`
- Sufficient AWS service limits for:
  - Lambda functions (5)
  - DynamoDB tables (2)
  - S3 buckets (2)
  - CloudFront distributions (1)

### Development Environment
- **Node.js**: Version 18 or higher
- **npm**: Version 8 or higher
- **Git**: For version control
- **Docker**: Optional, for local Lambda testing

### Required Permissions
Your AWS user/role needs permissions for:
- CloudFormation stack operations
- Lambda function management
- DynamoDB table operations
- S3 bucket operations
- IAM role creation
- API Gateway management
- CloudFront distribution management

### Domain Requirements (Production/Staging)
- Domain registered and managed through Cloudflare
- Cloudflare account with domain management access
- SSL certificate configured in AWS Certificate Manager

## 🛠 Environment Setup

### 1. Clone Repository
```bash
git clone <your-repository-url>
cd boat_listing_project
```

### 2. Install Dependencies

**Backend**:
```bash
cd backend
npm install
```

**Frontend**:
```bash
cd ../frontend
npm install
```

**Infrastructure**:
```bash
cd ../infrastructure
npm install
```

### 3. Configure AWS Credentials
```bash
aws configure
# Enter your AWS Access Key ID, Secret Access Key, and region (us-east-1 recommended)
```

### 4. Bootstrap CDK (First Time Only)
```bash
cd infrastructure
cdk bootstrap
```

## 🚀 Deployment Procedures

### Production Deployment (With Custom Domains)

#### Step 1: Configure Custom Domains

**⚠️ REQUIRED**: Set up custom domains before deployment.

1. **Add Domain to Cloudflare**
   - Add your domain to Cloudflare dashboard
   - Update nameservers at your domain registrar
   - Wait for DNS propagation (up to 24 hours)

2. **Generate Origin Certificate**
   - Go to Cloudflare SSL/TLS → Origin Server
   - Create certificate for your domain and API subdomain
   - Download certificate and private key

3. **Import Certificate to AWS ACM**
   ```bash
   aws acm import-certificate \
     --certificate fileb://certificate.pem \
     --private-key fileb://private-key.pem \
     --region us-east-1
   ```
   - Note the Certificate ARN for infrastructure configuration

4. **Update Infrastructure Configuration**
   ```typescript
   // In infrastructure/lib/boat-listing-stack.ts
   const certificateArn = 'arn:aws:acm:us-east-1:ACCOUNT:certificate/CERT-ID';
   ```

#### Step 2: Build Application Components

**Build Backend Services**:
```bash
cd backend
npm run build
```

This creates Lambda deployment packages in `backend/dist/`:
- `auth-service.zip`
- `listing-service.zip`
- `search-service.zip`
- `media-service.zip`
- `email-service.zip`

**Build Frontend Application**:
```bash
cd frontend
npm run build
```

This creates optimized frontend assets in `frontend/dist/`.

#### Step 3: Deploy Infrastructure

```bash
cd infrastructure

# Production deployment (default - uses custom domains)
npm run deploy:prod

# Or manually with CDK
cdk deploy --context environment=prod --context useCustomDomains=true --require-approval never
```

**Expected Output with Custom Domains**:
```
✅  BoatListingStack-prod

Outputs:
BoatListingStack-prod.ApiUrl = https://api.harborlist.com (development environment) (development environment) (development environment)
BoatListingStack-prod.FrontendUrl = https://harborlist.com (development environment) (development environment) (development environment)
BoatListingStack-prod.MediaBucketName = boat-listing-media-prod-674191656375
BoatListingStack-prod.ApiGatewayCustomDomain = api.harborlist.com
BoatListingStack-prod.TunnelInstanceId = i-1234567890abcdef0
```

#### Step 4: Configure Cloudflare DNS

After deployment, add these DNS records in Cloudflare:

```
# API Gateway
Type: CNAME
Name: api
Target: [API Gateway Custom Domain from outputs]
Proxy: ✅ Proxied

# Frontend (handled by Cloudflare Tunnel)
Type: CNAME
Name: @
Target: [tunnel-id].cfargotunnel.com
Proxy: ✅ Proxied
```

#### Step 5: Verify Production Deployment

```bash
# Test frontend
curl -I https://harborlist.com (development environment) (development environment) (development environment)
# Expected: HTTP/2 200

# Test API
curl https://api.harborlist.com/listings (development environment) (development environment) (development environment)
# Expected: {"listings":[],"total":0}

# Test admin panel
curl -I https://harborlist.com/admin/login (development environment) (development environment) (development environment)
# Expected: HTTP/2 200
```

### Staging Deployment

#### Step 1: Configure Staging Domains

Set up staging subdomains:
- **Frontend**: `staging.harborlist.com`
- **API**: `api-staging.harborlist.com`

#### Step 2: Deploy Staging Environment

```bash
cd infrastructure

# Staging deployment
npm run deploy:staging

# Or manually with CDK
cdk deploy --context environment=staging --context useCustomDomains=true --require-approval never
```

#### Step 3: Configure Staging DNS

Add DNS records for staging environment:

```
# Staging API
Type: CNAME
Name: api-staging
Target: [Staging API Gateway Domain]
Proxy: ✅ Proxied

# Staging Frontend
Type: CNAME
Name: staging
Target: [staging-tunnel-id].cfargotunnel.com
Proxy: ✅ Proxied
```

### Development Deployment (Without Custom Domains)

⚠️ **Only for local development and testing**

#### Step 1: Build Services
```bash
cd backend && npm run build
cd ../frontend && npm run build
```

#### Step 2: Deploy Without Custom Domains
```bash
cd infrastructure

# Development deployment without custom domains
npm run deploy:dev

# Or manually with CDK
cdk deploy --context environment=dev --context useCustomDomains=false --require-approval never
```

**Expected Output without Custom Domains**:
```
✅  BoatListingStack-dev

Outputs:
BoatListingStack-dev.ApiUrl = https://kz82y80qu2.execute-api.us-east-1.amazonaws.com/prod/ (development environment) (development environment) (development environment)
BoatListingStack-dev.FrontendUrl = https://dunxywperij31.cloudfront.net (development environment) (development environment) (development environment)
BoatListingStack-dev.MediaBucketName = boat-listing-media-dev-674191656375
```

#### Step 3: Verify Development Deployment

```bash
# Test frontend
curl -I https://dunxywperij31.cloudfront.net (development environment) (development environment) (development environment)
# Expected: HTTP/2 200

# Test API
curl https://kz82y80qu2.execute-api.us-east-1.amazonaws.com/prod/listings (development environment) (development environment) (development environment)
# Expected: {"listings":[],"total":0}
```

## 🔧 Environment-Specific Configuration

### Environment Variables

The CDK automatically configures these environment variables for Lambda functions:

**All Environments**:
- `USERS_TABLE`: DynamoDB users table name
- `LISTINGS_TABLE`: DynamoDB listings table name
- `JWT_SECRET`: JWT signing secret
- `AWS_REGION`: Deployment region

**Media Function**:
- `MEDIA_BUCKET`: S3 media bucket name

**Email Function**:
- `FROM_EMAIL`: SES verified sender email

### Frontend Configuration

Environment-specific configuration is handled automatically:

```bash
# Production
VITE_API_URL=https://api.harborlist.com (development environment) (development environment) (development environment)
VITE_ENVIRONMENT=production

# Staging
VITE_API_URL=https://api-staging.harborlist.com (development environment) (development environment) (development environment)
VITE_ENVIRONMENT=staging

# Development
VITE_API_URL=https://api-dev.harborlist.com (development environment) (development environment) (development environment)
VITE_ENVIRONMENT=development
```

## 📊 Post-Deployment Setup

### 1. Create Admin User

After successful deployment, create your first admin user:

```bash
cd backend

# Set environment variables
export USERS_TABLE=boat-users-prod  # or appropriate table name
export AWS_REGION=us-east-1

# Create super admin
npm run create-admin -- --email admin@yourcompany.com --name "Super Admin" --role super_admin

# Save the generated password securely!
```

### 2. Configure Monitoring

Set up CloudWatch dashboards and alarms:

```bash
# Create custom dashboard
aws cloudwatch put-dashboard \
  --dashboard-name "BoatListing-Production" \
  --dashboard-body file://monitoring/dashboard.json

# Set up billing alerts
aws cloudwatch put-metric-alarm \
  --alarm-name "HighBilling" \
  --alarm-description "Alert when billing exceeds threshold" \
  --metric-name EstimatedCharges \
  --namespace AWS/Billing \
  --statistic Maximum \
  --period 86400 \
  --threshold 100 \
  --comparison-operator GreaterThanThreshold
```

### 3. Verify All Services

Run comprehensive verification:

```bash
# Test all endpoints
curl -I https://harborlist.com (development environment) (development environment) (development environment)
curl -I https://api.harborlist.com/health (development environment) (development environment) (development environment)
curl -I https://harborlist.com/admin/login (development environment) (development environment) (development environment)

# Check database connectivity
aws dynamodb describe-table --table-name boat-users-prod
aws dynamodb describe-table --table-name boat-listings-prod

# Verify S3 buckets
aws s3 ls boat-listing-media-prod-*
```

## 🔄 Updates & Maintenance

### Code Updates

1. **Make Changes to Source Code**
2. **Rebuild Affected Components**:
   ```bash
   # Backend changes
   cd backend && npm run build
   
   # Frontend changes
   cd frontend && npm run build
   ```
3. **Redeploy**:
   ```bash
   cd infrastructure && cdk deploy --context environment=prod
   ```

### Infrastructure Updates

1. **Modify CDK Stack** in `../../infrastructure/lib/boat-listing-stack.ts`
2. **Preview Changes**:
   ```bash
   cd infrastructure && cdk diff --context environment=prod
   ```
3. **Deploy Changes**:
   ```bash
   cdk deploy --context environment=prod
   ```

### Database Schema Updates

DynamoDB is schemaless, but for major changes:
1. Create migration scripts in `backend/scripts/migrations/`
2. Test in development environment first
3. Apply during low-traffic periods
4. Monitor for issues post-migration

### Rolling Updates

For zero-downtime deployments:

1. **Deploy to Staging First**
2. **Run Integration Tests**
3. **Deploy to Production During Low Traffic**
4. **Monitor Metrics Post-Deployment**
5. **Rollback if Issues Detected**

## 🧹 Cleanup & Rollback

### Remove All Resources

```bash
cd infrastructure

# Remove production environment (implementation status) (implementation status) (implementation status)
cdk destroy --context environment=prod

# Remove staging environment  
cdk destroy --context environment=staging

# Remove development environment
cdk destroy --context environment=dev
```

**This will delete**:
- All Lambda functions
- DynamoDB tables (and all data)
- S3 buckets (and all files)
- CloudFront distribution
- API Gateway
- IAM roles and policies

### Rollback Deployment

#### Quick Rollback (Same Infrastructure)

```bash
# Rollback to previous code version
git checkout [PREVIOUS_COMMIT]

# Rebuild and redeploy
cd backend && npm run build
cd ../frontend && npm run build
cd ../infrastructure && cdk deploy --context environment=prod
```

#### Full Infrastructure Rollback

```bash
# Rollback infrastructure changes
git checkout [PREVIOUS_INFRASTRUCTURE_COMMIT]

# Deploy previous infrastructure
cd infrastructure
cdk deploy --context environment=prod
```

#### Emergency Rollback

For critical issues:

1. **Immediate DNS Failover**:
   - Point DNS to maintenance page
   - Update Cloudflare DNS records

2. **Assess and Fix**:
   - Check AWS service health
   - Review CloudWatch logs
   - Identify root cause

3. **Recovery Options**:
   - Restart services
   - Deploy hotfix
   - Full rollback to previous version

## 🔧 Troubleshooting

### Common Deployment Issues

#### CDK Bootstrap Error
```bash
# Solution: Bootstrap CDK in your region
cdk bootstrap aws://ACCOUNT-NUMBER/us-east-1
```

#### Lambda Handler Not Found
- Verify handler paths in CDK stack
- Ensure build process completed successfully
- Check Lambda function configuration in AWS Console

#### DynamoDB Permission Errors
- Verify IAM roles have correct permissions
- Check resource ARNs in CDK stack
- Ensure tables exist in correct region

#### CloudFront Distribution Issues
- Wait 15-20 minutes for global propagation
- Check Origin Access Control configuration
- Verify S3 bucket policies

#### Custom Domain Issues
- Verify certificate is imported in correct region (us-east-1)
- Check DNS propagation with `dig` or `nslookup`
- Verify Cloudflare proxy settings

### Debug Commands

**Check CDK Differences**:
```bash
cdk diff --context environment=prod
```

**View CloudFormation Events**:
```bash
aws cloudformation describe-stack-events --stack-name BoatListingStack-prod
```

**Test Lambda Functions Locally**:
```bash
cd backend
sam local start-api --template ../infrastructure/cdk.out/BoatListingStack-prod.template.json
```

**Check Service Health**:
```bash
# API Gateway health
curl https://api.harborlist.com/health (development environment) (development environment) (development environment)

# Lambda function logs
aws logs filter-log-events \
  --log-group-name "/aws/lambda/BoatListingStack-prod-ListingFunction" \
  --start-time $(date -d '1 hour ago' +%s)000
```

## 📈 Performance Optimization

### Lambda Optimization
- Use provisioned concurrency for consistent performance
- Optimize memory allocation based on usage patterns
- Implement connection pooling for DynamoDB
- Enable X-Ray tracing for performance insights

### DynamoDB Optimization
- Use Global Secondary Indexes for query patterns
- Implement proper partition key distribution
- Monitor hot partitions and throttling
- Use DynamoDB Accelerator (DAX) for read-heavy workloads

### CloudFront Optimization
- Configure appropriate cache behaviors
- Use compression for static assets
- Implement cache invalidation strategies
- Monitor cache hit ratios

### Cloudflare Optimization
- Enable Cloudflare caching rules
- Use Cloudflare Workers for edge computing
- Implement rate limiting and DDoS protection
- Monitor Cloudflare analytics

## 🔒 Security Best Practices

### Production Security Checklist

- [ ] Use AWS Secrets Manager for JWT secrets
- [ ] Enable CloudTrail for audit logging
- [ ] Configure WAF for API Gateway
- [ ] Set up VPC endpoints for private communication
- [ ] Enable S3 bucket encryption
- [ ] Configure proper CORS policies
- [ ] Implement rate limiting
- [ ] Set up monitoring and alerting
- [ ] Enable MFA for admin accounts
- [ ] Regular security audits and penetration testing

### SSL/TLS Configuration

- [ ] Use TLS 1.2 minimum
- [ ] Configure HSTS headers
- [ ] Implement certificate pinning
- [ ] Regular certificate rotation
- [ ] Monitor certificate expiration

### Access Control

- [ ] Implement least privilege access
- [ ] Use IAM roles instead of users
- [ ] Regular access reviews
- [ ] Enable AWS Config for compliance
- [ ] Implement resource-based policies

## 📞 Support & Escalation

### Deployment Support

1. **Level 1**: Development team member
2. **Level 2**: Senior developer/DevOps engineer  
3. **Level 3**: Technical lead/Manager
4. **Level 4**: External support (AWS, Cloudflare)

### Emergency Contacts

- **AWS Support**: Available through AWS Console
- **Cloudflare Support**: Available through Cloudflare Dashboard
- **Internal On-Call**: [Configure based on team structure]

### Documentation Updates

This deployment guide should be updated:
- After any infrastructure changes
- Following deployment issues and resolutions
- After procedure improvements
- During quarterly review cycles

---

**🎉 Your boat listing platform is successfully implemented and ready for production use!**

**Live URLs**:
- **Production Frontend**: https://harborlist.com (development environment) (development environment) (development environment)
- **Production API**: https://api.harborlist.com (development environment) (development environment) (development environment)
- **Admin Panel**: https://harborlist.com/admin/login (development environment) (development environment) (development environment)