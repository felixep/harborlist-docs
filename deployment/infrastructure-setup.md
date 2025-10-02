# Infrastructure Setup Guide

## Overview

This guide provides comprehensive instructions for setting up and deploying the MarineMarket infrastructure using AWS CDK. The platform uses a serverless architecture with AWS Lambda, DynamoDB, S3, and API Gateway, orchestrated through Infrastructure as Code (IaC) principles.

## Architecture Components

### Core Infrastructure Stack

The main infrastructure is defined in [`infrastructure/lib/boat-listing-stack.ts`](frontend/src/../../infrastructure/lib/boat-listing-stack.ts) and includes:

- **Compute**: AWS Lambda functions for serverless backend services
- **Storage**: DynamoDB tables for data persistence and S3 buckets for media storage
- **API**: API Gateway for RESTful API endpoints
- **CDN**: CloudFront distribution for frontend delivery
- **Security**: IAM roles, policies, and AWS Secrets Manager
- **Monitoring**: CloudWatch alarms, dashboards, and SNS notifications

### Supporting Constructs

- **Monitoring Construct** ([`infrastructure/lib/monitoring-construct.ts`](frontend/src/../../infrastructure/lib/monitoring-construct.ts)): CloudWatch monitoring and alerting
- **Cloudflare Tunnel Construct** ([`infrastructure/lib/cloudflare-tunnel-construct.ts`](frontend/src/../../infrastructure/lib/cloudflare-tunnel-construct.ts)): Secure tunnel infrastructure

## Prerequisites

### Required Tools

```bash
# Node.js and npm
node --version  # v18.0.0 or higher
npm --version   # v8.0.0 or higher

# AWS CLI
aws --version   # v2.0.0 or higher

# AWS CDK CLI
cdk --version   # v2.100.0 or higher
```

### AWS Account Setup

1. **AWS Account**: Administrative access to an AWS account
2. **AWS CLI Configuration**:
   ```bash
   aws configure
   # Enter: Access Key ID, Secret Access Key, Region (us-east-1), Output format (json)
   ```
3. **CDK Bootstrap** (first-time setup):
   ```bash
   cdk bootstrap aws://ACCOUNT-NUMBER/us-east-1
   ```

### Service Limits Verification

Ensure your AWS account has sufficient service limits:

- **Lambda Functions**: 5+ concurrent executions
- **DynamoDB Tables**: 2+ tables per region
- **S3 Buckets**: 2+ buckets
- **CloudFront Distributions**: 1+ distribution
- **API Gateway**: 1+ REST API

### Domain Requirements (Production)

**⚠️ IMPORTANT**: MarineMarket is designed to work with custom domains by default. This is not optional for production deployments.

#### Why Custom Domains Are Required

1. **🔒 Security**: Cloudflare provides DDoS protection, SSL termination, and security features
2. **🚀 Performance**: Global CDN with edge caching for better user experience
3. **🛡️ Origin Protection**: API Gateway and S3 are protected behind Cloudflare
4. **📊 Analytics**: Cloudflare provides detailed traffic analytics and monitoring

#### When to Use Custom Domains (Recommended)

✅ **Production deployments**  
✅ **Staging environments**  
✅ **Demo environments**  
✅ **Any public-facing deployment**

#### When You Can Skip Custom Domains

❌ **Local development only** (use `useCustomDomains=false`)  
❌ **Internal testing** (temporary setups)  
❌ **CI/CD pipeline testing** (automated tests)

⚠️ **Warning**: Without custom domains, you'll lose security features, performance benefits, and the frontend may not work correctly due to CORS and routing configurations.

**📖 Complete Setup Instructions**: See [Domain Setup Guide](domain-setup.md) for detailed domain configuration.

## Quick Start Deployment

### 1. Clone and Setup
```bash
# Clone the repository
git clone <repository-url>
cd boat_listing_project

# Install infrastructure dependencies
cd infrastructure
npm install

# Install backend dependencies
cd ../backend
npm install

# Install frontend dependencies
cd ../frontend
npm install
   cd infrastructure
   cdk bootstrap
   ```

### Required AWS Permissions

Your AWS user/role needs the following permissions:
- CloudFormation: Full access for stack operations
- Lambda: Create, update, delete functions and layers
- DynamoDB: Create, update, delete tables and indexes
- S3: Create, update, delete buckets and objects
- API Gateway: Create, update, delete APIs and resources
- IAM: Create, update, delete roles and policies
- CloudWatch: Create, update, delete alarms and dashboards
- Secrets Manager: Create, update, delete secrets
- SNS: Create, update, delete topics and subscriptions

## Environment Configuration

### Environment Types

The infrastructure supports three environments:
- **dev**: Development environment with minimal resources
- **staging**: Pre-production environment with production-like configuration
- **prod**: Production environment with full monitoring and security

### Environment-Specific Configuration

Environment configuration is handled through CDK context parameters:

```typescript
// infrastructure/lib/boat-listing-stack.ts (lines 15-20)
interface BoatListingStackProps extends cdk.StackProps {
  environment: 'dev' | 'staging' | 'prod';
  domainName?: string;
  apiDomainName?: string;
  alertEmail?: string;
}
```

### Custom Domain Configuration

For production deployments, configure custom domains:

1. **Import Cloudflare Origin Certificate** (lines 25-30):
   ```typescript
   let certificate: certificatemanager.ICertificate | undefined;
   
   if (domainName && apiDomainName) {
     certificate = certificatemanager.Certificate.fromCertificateArn(
       this, 'CloudflareOriginCertificate', 
       'arn:aws:acm:us-east-1:676032292155:certificate/93fe820b-e1bc-445c-8ab2-5a994cd95ed7'
     );
   }
   ```

2. **API Gateway Custom Domain** (lines 280-290):
   ```typescript
   let apiDomainNameResource: apigateway.DomainName | undefined;
   
   if (certificate && apiDomainName) {
     apiDomainNameResource = new apigateway.DomainName(this, 'ApiDomainName', {
       domainName: apiDomainName,
       certificate: certificate,
       endpointType: apigateway.EndpointType.REGIONAL,
     });
   }
   ```

## Database Setup

### DynamoDB Tables

The infrastructure creates five DynamoDB tables with pay-per-request billing:

#### 1. Listings Table (lines 32-42)
```typescript
const listingsTable = new dynamodb.Table(this, 'ListingsTable', {
  tableName: 'boat-listings',
  partitionKey: { name: 'id', type: dynamodb.AttributeType.STRING },
  billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
  removalPolicy: cdk.RemovalPolicy.DESTROY,
});

listingsTable.addGlobalSecondaryIndex({
  indexName: 'location-index',
  partitionKey: { name: 'location', type: dynamodb.AttributeType.STRING },
  sortKey: { name: 'createdAt', type: dynamodb.AttributeType.STRING },
});
```

#### 2. Users Table (lines 44-54)
```typescript
const usersTable = new dynamodb.Table(this, 'UsersTable', {
  tableName: 'boat-users',
  partitionKey: { name: 'id', type: dynamodb.AttributeType.STRING },
  billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
  removalPolicy: cdk.RemovalPolicy.DESTROY,
});

usersTable.addGlobalSecondaryIndex({
  indexName: 'email-index',
  partitionKey: { name: 'email', type: dynamodb.AttributeType.STRING },
});
```

#### 3. Audit Logs Table (lines 56-82)
```typescript
const auditLogsTable = new dynamodb.Table(this, 'AuditLogsTable', {
  tableName: 'boat-audit-logs',
  partitionKey: { name: 'id', type: dynamodb.AttributeType.STRING },
  sortKey: { name: 'timestamp', type: dynamodb.AttributeType.STRING },
  billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
  removalPolicy: cdk.RemovalPolicy.DESTROY,
  timeToLiveAttribute: 'ttl', // For automatic cleanup of old logs
});
```

#### 4. Admin Sessions Table (lines 84-96)
```typescript
const adminSessionsTable = new dynamodb.Table(this, 'AdminSessionsTable', {
  tableName: 'boat-admin-sessions',
  partitionKey: { name: 'sessionId', type: dynamodb.AttributeType.STRING },
  billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
  removalPolicy: cdk.RemovalPolicy.DESTROY,
  timeToLiveAttribute: 'expiresAt',
});
```

#### 5. Login Attempts Table (lines 104-122)
```typescript
const loginAttemptsTable = new dynamodb.Table(this, 'LoginAttemptsTable', {
  tableName: 'boat-login-attempts',
  partitionKey: { name: 'id', type: dynamodb.AttributeType.STRING },
  sortKey: { name: 'timestamp', type: dynamodb.AttributeType.STRING },
  billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
  removalPolicy: cdk.RemovalPolicy.DESTROY,
  timeToLiveAttribute: 'ttl',
});
```

### Global Secondary Indexes (GSI)

Each table includes strategically designed GSIs for efficient querying:
- **Listings**: Location-based queries with timestamp sorting
- **Users**: Email-based authentication lookups
- **Audit Logs**: User, action, and resource-based queries
- **Admin Sessions**: User-based session management
- **Login Attempts**: Email and IP-based security monitoring

## Storage Configuration

### S3 Buckets

#### Media Bucket (lines 124-132)
```typescript
const mediaBucket = new s3.Bucket(this, 'MediaBucket', {
  bucketName: `boat-listing-media-${this.account}`,
  cors: [{
    allowedMethods: [s3.HttpMethods.GET, s3.HttpMethods.POST, s3.HttpMethods.PUT],
    allowedOrigins: ['*'],
    allowedHeaders: ['*'],
  }],
  removalPolicy: cdk.RemovalPolicy.DESTROY,
});
```

#### Frontend Bucket (lines 134-150)
```typescript
const frontendBucket = new s3.Bucket(this, 'FrontendBucket', {
  bucketName: `boat-listing-frontend-${this.account}`,
  removalPolicy: cdk.RemovalPolicy.DESTROY,
  websiteIndexDocument: 'index.html',
  websiteErrorDocument: 'index.html', // SPA routing support
  blockPublicAccess: new s3.BlockPublicAccess({
    blockPublicAcls: false,
    ignorePublicAcls: false,
    blockPublicPolicy: false,
    restrictPublicBuckets: false,
  }),
});
```

## Lambda Functions

### Function Configuration

All Lambda functions use Node.js 18.x runtime with environment-specific configuration:

#### Auth Function (lines 158-165)
```typescript
const authFunction = new lambda.Function(this, 'AuthFunction', {
  runtime: lambda.Runtime.NODEJS_18_X,
  handler: 'auth-service/index.handler',
  code: lambda.Code.fromAsset('../backend/dist/packages/auth-service.zip'),
  environment: {
    USERS_TABLE: usersTable.tableName,
    JWT_SECRET: 'your-jwt-secret-key', // Use AWS Secrets Manager in production
  },
});
```

#### Admin Function (lines 200-220)
```typescript
const adminFunction = new lambda.Function(this, 'AdminFunction', {
  runtime: lambda.Runtime.NODEJS_18_X,
  handler: 'admin-service/index.handler',
  code: lambda.Code.fromAsset('../backend/dist/packages/admin-service.zip'),
  timeout: cdk.Duration.seconds(30),
  memorySize: 512,
  environment: {
    LISTINGS_TABLE: listingsTable.tableName,
    USERS_TABLE: usersTable.tableName,
    AUDIT_LOGS_TABLE: auditLogsTable.tableName,
    ADMIN_SESSIONS_TABLE: adminSessionsTable.tableName,
    LOGIN_ATTEMPTS_TABLE: loginAttemptsTable.tableName,
    JWT_SECRET_ARN: jwtSecret.secretArn,
    ENVIRONMENT: environment,
    NODE_ENV: environment === 'prod' ? 'production' : 'development',
  },
});
```

### Function Permissions

IAM permissions are granted using CDK's built-in methods:

```typescript
// Database permissions (lines 225-235)
usersTable.grantReadWriteData(authFunction);
listingsTable.grantReadWriteData(listingFunction);
usersTable.grantReadWriteData(listingFunction);

// S3 permissions
mediaBucket.grantReadWrite(mediaFunction);

// Secrets Manager permissions
jwtSecret.grantRead(adminFunction);
```

## API Gateway Configuration

### REST API Setup (lines 270-278)
```typescript
const api = new apigateway.RestApi(this, 'BoatListingApi', {
  restApiName: 'Boat Listing API',
  defaultCorsPreflightOptions: {
    allowOrigins: ['*'],
    allowMethods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
    allowHeaders: [
      'Content-Type', 'Authorization', 'X-Amz-Date',
      'X-Api-Key', 'X-Amz-Security-Token', 'X-Amz-User-Agent'
    ],
  },
});
```

### API Routes

The API includes comprehensive routes for all services:

```typescript
// Listings routes
const listings = api.root.addResource('listings');
listings.addMethod('GET', new apigateway.LambdaIntegration(listingFunction));
listings.addMethod('POST', new apigateway.LambdaIntegration(listingFunction));

// Authentication routes
const auth = api.root.addResource('auth');
const login = auth.addResource('login');
login.addMethod('POST', new apigateway.LambdaIntegration(authFunction));

// Admin routes with proxy
const admin = api.root.addResource('admin');
const adminProxy = admin.addResource('{proxy+}');
adminProxy.addMethod('ANY', new apigateway.LambdaIntegration(adminFunction));
```

## Security Configuration

### JWT Secret Management (lines 190-199)
```typescript
const jwtSecret = new secretsmanager.Secret(this, 'AdminJwtSecret', {
  secretName: `boat-listing-admin-jwt-${environment}`,
  description: 'JWT secret for admin authentication',
  generateSecretString: {
    secretStringTemplate: JSON.stringify({ username: 'admin' }),
    generateStringKey: 'password',
    excludeCharacters: '"@/\\',
    passwordLength: 32,
  },
});
```

### IAM Policies

Custom IAM policies for enhanced security:

```typescript
// CloudWatch permissions for admin function
adminFunction.addToRolePolicy(new iam.PolicyStatement({
  actions: [
    'cloudwatch:PutMetricData',
    'logs:CreateLogGroup',
    'logs:CreateLogStream',
    'logs:PutLogEvents',
  ],
  resources: ['*'],
}));
```

## Deployment Commands

### Package.json Scripts

The infrastructure includes comprehensive deployment scripts in [`infrastructure/package.json`](frontend/src/../../infrastructure/package.json):

```json
{
  "scripts": {
    "deploy:dev": "cdk deploy -c environment=dev",
    "deploy:staging": "cdk deploy -c environment=staging", 
    "deploy:prod": "cdk deploy -c environment=prod",
    "deploy:dev:no-domains": "cdk deploy -c environment=dev -c useCustomDomains=false",
    "synth:dev": "cdk synth -c environment=dev",
    "diff:dev": "cdk diff -c environment=dev"
  }
}
```

### Deployment Process

#### 1. Build Backend Services
```bash
cd backend
npm install
npm run build
```

#### 2. Build Frontend Application
```bash
cd frontend
npm install
npm run build
```

#### 3. Deploy Infrastructure
```bash
cd infrastructure
npm install

# Development deployment
npm run deploy:dev

# Production deployment with custom domains
npm run deploy:prod

# Development without custom domains
npm run deploy:dev:no-domains
```

### Deployment Script

Use the automated deployment script [`infrastructure/deploy.sh`](../../infrastructure/deploy.sh):

```bash
# Production deployment with domains
./deploy.sh prod

# Development deployment without domains
./deploy.sh dev false
```

## Environment-Specific Configurations

### Development Environment
- Minimal resources for cost optimization
- Relaxed security for development convenience
- Debug logging enabled
- No custom domains required

### Staging Environment
- Production-like configuration
- Full monitoring and alerting
- Custom domains recommended
- Performance testing enabled

### Production Environment
- Full security hardening
- Comprehensive monitoring
- Custom domains required
- Backup and disaster recovery
- Cost optimization strategies

## Monitoring and Alerting Setup

The infrastructure includes comprehensive monitoring through the MonitoringConstruct:

### CloudWatch Alarms (lines 350-380)
```typescript
// Admin Function Error Rate Alarm
const adminErrorAlarm = new cloudwatch.Alarm(this, 'AdminFunctionErrorAlarm', {
  alarmName: `admin-function-errors-${environment}`,
  alarmDescription: 'High error rate in admin Lambda function',
  metric: adminFunction.metricErrors({
    period: cdk.Duration.minutes(5),
    statistic: 'Sum',
  }),
  threshold: 5,
  evaluationPeriods: 2,
});
```

### SNS Notifications (lines 340-348)
```typescript
const adminAlertTopic = new sns.Topic(this, 'AdminServiceAlerts', {
  topicName: `admin-service-alerts-${environment}`,
  displayName: `Admin Service Monitoring Alerts - ${environment}`,
});

if (alertEmail) {
  adminAlertTopic.addSubscription(
    new subscriptions.EmailSubscription(alertEmail)
  );
}
```

### CloudWatch Dashboard (lines 430-480)
```typescript
const adminDashboard = new cloudwatch.Dashboard(this, 'AdminServiceDashboard', {
  dashboardName: `admin-service-${environment}`,
});

adminDashboard.addWidgets(
  new cloudwatch.GraphWidget({
    title: 'Admin Function Performance',
    left: [adminFunction.metricInvocations(), adminFunction.metricErrors()],
    right: [adminFunction.metricDuration()],
  })
);
```

## Stack Outputs

The infrastructure provides comprehensive outputs for integration:

```typescript
// API and Frontend URLs
new cdk.CfnOutput(this, 'ApiUrl', {
  value: apiDomainNameResource ? `https://${apiDomainName}` : api.url,
  description: 'API Gateway URL',
});

// Resource Names
new cdk.CfnOutput(this, 'MediaBucketName', {
  value: mediaBucket.bucketName,
  description: 'S3 Media Bucket Name',
});

// Admin Infrastructure
new cdk.CfnOutput(this, 'AdminFunctionArn', {
  value: adminFunction.functionArn,
  description: 'Admin Lambda Function ARN',
});
```

## Troubleshooting

### Common Issues

1. **CDK Bootstrap Error**:
   ```bash
   cdk bootstrap aws://ACCOUNT-NUMBER/REGION
   ```

2. **Lambda Handler Not Found**:
   - Verify build process completed: `npm run build` in backend
   - Check handler paths in CDK stack configuration

3. **DynamoDB Permission Errors**:
   - Verify IAM roles have correct permissions
   - Check resource ARNs in grant statements

4. **Custom Domain Issues**:
   - Ensure certificate is imported in ACM
   - Verify domain configuration in Cloudflare
   - Check DNS propagation (can take up to 48 hours)

### Debug Commands

```bash
# View CDK differences
cdk diff -c environment=dev

# Synthesize CloudFormation template
cdk synth -c environment=dev

# View CloudFormation events
aws cloudformation describe-stack-events --stack-name BoatListingStack-dev

# Check Lambda function logs
aws logs filter-log-events \
  --log-group-name "/aws/lambda/BoatListingStack-dev-AdminFunction" \
  --region us-east-1
```

## Cost Optimization

### Resource Optimization Strategies

1. **DynamoDB**: Pay-per-request billing for variable workloads
2. **Lambda**: Right-sized memory allocation based on usage patterns
3. **S3**: Lifecycle policies for media storage optimization
4. **CloudWatch**: Retention policies for log management

### Cost Monitoring

Set up billing alerts and use AWS Cost Explorer for:
- Monthly spend tracking
- Service-level cost analysis
- Usage pattern optimization
- Resource utilization monitoring

## Security Best Practices

### Production Security Checklist

- [ ] Use AWS Secrets Manager for all secrets
- [ ] Enable CloudTrail for audit logging
- [ ] Configure WAF for API Gateway protection
- [ ] Implement VPC endpoints for private communication
- [ ] Enable S3 bucket encryption at rest
- [ ] Configure proper CORS policies
- [ ] Implement API rate limiting
- [ ] Set up comprehensive monitoring and alerting
- [ ] Regular security audits and penetration testing

### IAM Best Practices

- Principle of least privilege for all roles
- Regular access review and cleanup
- Use managed policies where possible
- Implement resource-based policies for cross-service access

## Next Steps

After successful infrastructure deployment:

1. **Configure Monitoring**: Set up CloudWatch dashboards and alerts
2. **Security Hardening**: Implement additional security measures for production
3. **Performance Testing**: Conduct load testing and optimization
4. **Backup Strategy**: Implement data backup and disaster recovery procedures
5. **Documentation**: Update operational runbooks and procedures

## Related Documentation

- [Monitoring and Alerting Guide](monitoring.md)
- [Operational Runbooks](../operations/)
- [Security Architecture](../security/)
- [Cost Optimization Guide](cost-optimization.md)