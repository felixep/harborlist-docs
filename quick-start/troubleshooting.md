# Troubleshooting Guide

## 🚨 Quick Diagnostic Commands

### Essential Health Checks

```bash
# 1. Test website accessibility
curl -I https://harborlist.com (development environment) (development environment) (development environment)
curl -I https://api.harborlist.com/health (development environment) (development environment) (development environment)

# 2. Check AWS infrastructure status
aws cloudformation describe-stacks --stack-name BoatListingStack-prod --query 'Stacks[0].StackStatus'

# 3. Test API endpoints
curl https://api.harborlist.com/listings (development environment) (development environment) (development environment)
curl -X POST https://api.harborlist.com/auth/login (development environment) (development environment) (development environment) -H "Content-Type: application/json" -d '{"email":"test@example.com","password":"test"}'

# 4. Check database connectivity
aws dynamodb describe-table --table-name boat-users-prod
aws dynamodb describe-table --table-name boat-listings-prod
```

### Environment-Specific Diagnostics

```bash
# Development Environment
curl -I https://dev.harborlist.com (development environment) (development environment) (development environment)
aws ssm start-session --target $(aws cloudformation describe-stacks --stack-name BoatListingStack-dev --query 'Stacks[0].Outputs[?OutputKey==`TunnelInstanceId`].OutputValue' --output text)

# Staging Environment
curl -I https://staging.harborlist.com (development environment) (development environment) (development environment)
curl -I https://api-staging.harborlist.com/health (development environment) (development environment) (development environment)

# Production Environment (implementation status) (implementation status) (implementation status)
curl -I https://harborlist.com (development environment) (development environment) (development environment)
curl -I https://api.harborlist.com/health (development environment) (development environment) (development environment)
```

## 🔧 Development Issues

### Frontend Development Problems

#### Port Already in Use

**Symptoms:**
- `npm run dev` fails with "EADDRINUSE" error
- Cannot start development server

**Solutions:**
```bash
# Find process using port 3000
lsof -ti:3000

# Kill process
lsof -ti:3000 | xargs kill -9

# Or use different port
npm run dev -- --port 3001
```

#### TypeScript Errors

**Symptoms:**
- Build fails with TypeScript errors
- IDE shows type errors

**Solutions:**
```bash
# Clear TypeScript cache
rm -rf node_modules/.cache
rm -rf .tsbuildinfo

# Reinstall dependencies
rm -rf node_modules package-lock.json
npm install

# Run type check
npm run type-check

# Fix common issues
npm run lint:fix
```

#### Build Issues

**Symptoms:**
- `npm run build` fails
- Missing dependencies or modules

**Solutions:**
```bash
# Clear all build artifacts
npm run clean

# Clear Vite cache (frontend)
cd frontend
rm -rf dist node_modules/.vite
npm run build

# Clear TypeScript build (backend)
cd backend
rm -rf dist
npm run build
```

#### API Integration Issues

**Symptoms:**
- CORS errors in browser console
- API requests failing
- Authentication errors

**Solutions:**
```bash
# Check API URL configuration
echo $VITE_API_URL

# Test API directly
curl -v https://api.harborlist.com/health (development environment) (development environment) (development environment)

# Debug authentication
const token = localStorage.getItem('authToken');
console.log('Token:', token);

# Check token expiration
if (token) {
  const payload = JSON.parse(atob(token.split('.')[1]));
  console.log('Expires:', new Date(payload.exp * 1000));
}
```

### Backend Development Problems

#### Lambda Function Errors

**Symptoms:**
- Functions fail to deploy
- Runtime errors in CloudWatch logs
- Handler not found errors

**Solutions:**
```bash
# Check build output
ls -la backend/dist/

# Test function locally
cd backend
sam local invoke ListingFunction --event test-events/sample.json

# Check CloudWatch logs
aws logs filter-log-events \
  --log-group-name "/aws/lambda/BoatListingStack-ListingFunction" \
  --start-time $(date -d '1 hour ago' +%s)000

# Verify handler configuration in CDK
grep -r "handler:" infrastructure/lib/
```

#### Database Connection Issues

**Symptoms:**
- DynamoDB operations fail
- Permission denied errors
- Table not found errors

**Solutions:**
```bash
# Check table exists
aws dynamodb describe-table --table-name boat-users-prod

# Test table access
aws dynamodb scan --table-name boat-users-prod --limit 1

# Check IAM permissions
aws iam get-role-policy --role-name BoatListingStack-ListingFunctionRole --policy-name DynamoDBAccess

# Verify environment variables
aws lambda get-function-configuration --function-name BoatListingStack-ListingFunction
```

## 🚀 Deployment Problems

### CDK Deployment Issues

#### Bootstrap Errors

**Symptoms:**
- CDK deploy fails with bootstrap error
- Missing CDK toolkit stack

**Solutions:**
```bash
# Bootstrap CDK in your region
cdk bootstrap aws://ACCOUNT-NUMBER/us-east-1

# Check bootstrap stack
aws cloudformation describe-stacks --stack-name CDKToolkit

# Re-bootstrap if needed
cdk bootstrap --force
```

#### Stack Deployment Failures

**Symptoms:**
- CloudFormation stack creation fails
- Resource creation errors
- Rollback occurs

**Solutions:**
```bash
# Check stack events
aws cloudformation describe-stack-events --stack-name BoatListingStack-prod

# View detailed error
aws cloudformation describe-stack-resources --stack-name BoatListingStack-prod

# Check CDK diff before deploy
cdk diff --context environment=prod

# Deploy with verbose logging
cdk deploy --context environment=prod --verbose
```

#### Lambda Deployment Issues

**Symptoms:**
- Lambda functions not updating
- Code changes not reflected
- Package size errors

**Solutions:**
```bash
# Rebuild Lambda packages
cd backend
./build.sh

# Check package sizes
ls -lh backend/dist/*.zip

# Force Lambda update
aws lambda update-function-code \
  --function-name BoatListingStack-ListingFunction \
  --zip-file fileb://backend/dist/listing-service.zip
```

### Custom Domain Issues

#### SSL Certificate Problems

**Symptoms:**
- SSL certificate warnings
- Certificate not found errors
- HTTPS not working

**Solutions:**
```bash
# Check certificate in ACM
aws acm list-certificates --region us-east-1

# Verify certificate is validated
aws acm describe-certificate --certificate-arn arn:aws:acm:us-east-1:ACCOUNT:certificate/CERT-ID

# Test SSL configuration
openssl s_client -connect api.harborlist.com:443 -servername api.harborlist.com

# Check Cloudflare SSL settings (via dashboard)
```

#### DNS Resolution Issues

**Symptoms:**
- Domain not resolving
- DNS propagation issues
- Wrong IP addresses

**Solutions:**
```bash
# Test DNS resolution
nslookup harborlist.com
dig harborlist.com

# Test from different DNS servers
nslookup harborlist.com 8.8.8.8
nslookup harborlist.com 1.1.1.1

# Check Cloudflare DNS settings
# (Via Cloudflare Dashboard → DNS)

# Clear local DNS cache
# macOS: sudo dscacheutil -flushcache
# Linux: sudo systemctl restart systemd-resolved
```

## 🌐 Infrastructure Issues

### Cloudflare Tunnel Problems

#### Tunnel Service Issues

**Symptoms:**
- Website returns 502/503 errors
- Tunnel shows as disconnected
- Service won't start

**Solutions:**
```bash
# Get EC2 instance ID
INSTANCE_ID=$(aws cloudformation describe-stacks \
  --stack-name BoatListingStack-dev \
  --query 'Stacks[0].Outputs[?OutputKey==`TunnelInstanceId`].OutputValue' \
  --output text)

# Connect to instance
aws ssm start-session --target $INSTANCE_ID

# Check tunnel service
sudo systemctl status cloudflared

# Restart tunnel service
sudo systemctl restart cloudflared

# Check logs
sudo journalctl -u cloudflared -n 20
```

#### Configuration Issues

**Symptoms:**
- Tunnel won't start
- Configuration errors in logs
- Wrong routing

**Solutions:**
```bash
# Check configuration file
sudo cat /etc/cloudflared/config.yml

# Validate YAML syntax
python3 -c "import yaml; yaml.safe_load(open('/etc/cloudflared/config.yml'))"

# Test configuration manually
sudo -u cloudflared cloudflared tunnel --config /etc/cloudflared/config.yml run [TUNNEL_ID]

# Check file permissions
sudo ls -la /etc/cloudflared/
sudo chmod 600 /etc/cloudflared/*.json
sudo chown cloudflared:cloudflared /etc/cloudflared/*
```

### S3 Access Issues

#### Bucket Access Denied

**Symptoms:**
- S3 returns 403 Forbidden
- Website content not loading
- VPC endpoint access issues

**Solutions:**
```bash
# Test S3 access from EC2
curl -I http://boat-listing-frontend-ACCOUNT.s3-website-us-east-1.amazonaws.com (development environment) (development environment) (development environment)

# Check bucket policy
aws s3api get-bucket-policy --bucket boat-listing-frontend-ACCOUNT

# Verify VPC endpoint
aws ec2 describe-vpc-endpoints --filters Name=service-name,Values=com.amazonaws.us-east-1.s3

# Update bucket policy with correct VPC endpoint ID
aws s3api put-bucket-policy --bucket BUCKET-NAME --policy file://bucket-policy.json
```

#### Website Hosting Issues

**Symptoms:**
- S3 website returns 404
- Index document not loading
- Routing issues

**Solutions:**
```bash
# Check website configuration
aws s3api get-bucket-website --bucket BUCKET-NAME

# Enable website hosting
aws s3api put-bucket-website --bucket BUCKET-NAME --website-configuration '{
  "IndexDocument": {"Suffix": "index.html"},
  "ErrorDocument": {"Key": "index.html"}
}'

# Check bucket contents
aws s3 ls s3://BUCKET-NAME/

# Redeploy frontend if needed
cd infrastructure
cdk deploy --context environment=prod
```

## 🔐 Authentication & Authorization Issues

### JWT Token Problems

**Symptoms:**
- Authentication fails
- Token expired errors
- Invalid token errors

**Solutions:**
```bash
# Check token in browser storage
localStorage.getItem('authToken')

# Decode JWT token
echo "TOKEN" | cut -d. -f2 | base64 -d | jq

# Test token with API
curl -H "Authorization: Bearer TOKEN" https://api.harborlist.com/listings (development environment) (development environment) (development environment)

# Clear token and re-authenticate
localStorage.removeItem('authToken')
```

### Admin Access Issues

**Symptoms:**
- Cannot access admin panel
- Admin login fails
- Permission denied errors

**Solutions:**
```bash
# Check admin user exists
aws dynamodb query \
  --table-name boat-users-prod \
  --index-name email-index \
  --key-condition-expression "email = :email" \
  --expression-attribute-values '{":email":{"S":"admin@example.com"}}'

# Create admin user if missing
cd backend
npm run create-admin -- --email admin@example.com --name "Admin" --role super_admin

# Reset admin password
npm run create-admin -- --email admin@example.com --name "Admin" --role super_admin --password "NewPassword123!"
```

## 📊 Performance Issues

### Slow Loading Times

**Symptoms:**
- Website loads slowly
- High response times
- Poor user experience

**Diagnosis:**
```bash
# Test response times
curl -w "@curl-format.txt" -o /dev/null -s https://harborlist.com (development environment) (development environment) (development environment)

# Check CloudFront cache hit ratio
aws cloudwatch get-metric-statistics \
  --namespace AWS/CloudFront \
  --metric-name CacheHitRate \
  --dimensions Name=DistributionId,Value=DISTRIBUTION-ID \
  --start-time $(date -u -d '24 hours ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 3600 \
  --statistics Average
```

**Solutions:**
- Enable Cloudflare caching and optimization
- Optimize images and static assets
- Use CDN for static content
- Implement lazy loading
- Optimize database queries

### High Resource Usage

**Symptoms:**
- High CPU/memory usage
- Lambda timeouts
- DynamoDB throttling

**Solutions:**
```bash
# Check Lambda metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Duration \
  --dimensions Name=FunctionName,Value=BoatListingStack-ListingFunction \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 300 \
  --statistics Average,Maximum

# Check DynamoDB metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/DynamoDB \
  --metric-name ConsumedReadCapacityUnits \
  --dimensions Name=TableName,Value=boat-listings-prod \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 300 \
  --statistics Sum
```

## 🔍 Monitoring & Debugging

### CloudWatch Logs Analysis

```bash
# View Lambda function logs
aws logs filter-log-events \
  --log-group-name "/aws/lambda/BoatListingStack-ListingFunction" \
  --start-time $(date -d '1 hour ago' +%s)000

# Search for errors
aws logs filter-log-events \
  --log-group-name "/aws/lambda/BoatListingStack-ListingFunction" \
  --filter-pattern "ERROR"

# View API Gateway logs
aws logs filter-log-events \
  --log-group-name "API-Gateway-Execution-Logs_API-ID/prod"
```

### Application Debugging

#### Frontend Debugging

```javascript
// Enable debug mode
localStorage.setItem('debug', 'true');

// Check React Query cache
import { useQueryClient } from '@tanstack/react-query';
const queryClient = useQueryClient();
console.log(queryClient.getQueryCache().getAll());

// Monitor API calls
// Open browser DevTools → Network tab
```

#### Backend Debugging

```typescript
// Add debug logging
console.log('Debug info:', { userId, requestId, data });

// Use X-Ray for tracing
import AWSXRay from 'aws-xray-sdk-core';
const segment = AWSXRay.getSegment();
segment.addAnnotation('userId', userId);
```

## 🚨 Emergency Recovery Procedures

### Complete Service Outage

**Immediate Actions:**

1. **Check AWS Service Health**
   - Visit: https://status.aws.amazon.com/ (development environment) (development environment) (development environment)
   - Check us-east-1 region status

2. **Quick Recovery Steps**
   ```bash
   # Restart critical services
   aws ec2 reboot-instances --instance-ids $INSTANCE_ID
   
   # Wait and test
   sleep 300
   curl -I https://harborlist.com (development environment) (development environment) (development environment)
   ```

3. **Emergency DNS Failover**
   - Point DNS to maintenance page
   - Update Cloudflare DNS records
   - Notify users via status page

### Database Recovery

**Data Loss Prevention:**
```bash
# Create immediate backup
aws dynamodb create-backup --table-name boat-users-prod --backup-name emergency-backup-$(date +%Y%m%d-%H%M%S)

# Check backup status
aws dynamodb describe-backup --backup-arn BACKUP-ARN
```

**Recovery from Backup:**
```bash
# Restore table from backup
aws dynamodb restore-table-from-backup \
  --target-table-name boat-users-prod-restored \
  --backup-arn BACKUP-ARN
```

### Configuration Rollback

**Infrastructure Rollback:**
```bash
# Rollback to previous CDK version
git checkout [PREVIOUS-COMMIT]
cd infrastructure
cdk deploy --context environment=prod
```

**Application Rollback:**
```bash
# Deploy previous application version
git checkout [PREVIOUS-APP-COMMIT]
cd backend && npm run build
cd ../frontend && npm run build
cd ../infrastructure && cdk deploy --context environment=prod
```

## 📞 Getting Help

### Internal Escalation Path

1. **Level 1**: Check this troubleshooting guide
2. **Level 2**: Review system logs and metrics
3. **Level 3**: Contact development team
4. **Level 4**: Escalate to DevOps/Infrastructure team

### External Support Resources

- **AWS Support**: Available through AWS Console
- **Cloudflare Support**: Available through Cloudflare Dashboard
- **Community Forums**: Stack Overflow, AWS Forums, Reddit
- **Documentation**: AWS Docs, Cloudflare Docs, React Docs

### Creating Support Tickets

**Information to Include:**
- Environment (dev/staging/prod)
- Error messages and logs
- Steps to reproduce
- Timeline of when issue started
- Impact on users
- Troubleshooting steps already tried

### Documentation Updates

**After Resolving Issues:**
1. Document the problem and solution
2. Update this troubleshooting guide
3. Share learnings with the team
4. Update monitoring and alerting
5. Create preventive measures

## 🛡️ Prevention Best Practices

### Regular Maintenance

**Daily:**
- Monitor system health dashboards
- Check error rates and response times
- Review critical alerts

**Weekly:**
- Review performance metrics
- Check system updates
- Verify backup integrity

**Monthly:**
- Update dependencies
- Review and optimize configurations
- Test disaster recovery procedures
- Security audit and updates

### Monitoring Setup

```bash
# Set up comprehensive monitoring
aws cloudwatch put-metric-alarm \
  --alarm-name "HighErrorRate" \
  --alarm-description "High error rate detected" \
  --metric-name 4xxErrorRate \
  --namespace AWS/ApiGateway \
  --statistic Average \
  --period 300 \
  --threshold 5 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2
```

### Health Checks

Create automated health check scripts:

```bash
#!/bin/bash
# health-check.sh

DOMAIN="harborlist.com"
API_DOMAIN="api.harborlist.com"

# Test website
if ! curl -f -s https://$DOMAIN (development environment) (development environment) (development environment) > /dev/null; then
    echo "ALERT: Website is down"
    # Send notification
fi

# Test API
if ! curl -f -s https://$API_DOMAIN/health (development environment) (development environment) (development environment) > /dev/null; then
    echo "ALERT: API is down"
    # Send notification
fi

echo "Health check completed"
```

---

**Last Updated:** $(date)
**Version:** 2.0
**Next Review:** $(date -d '+3 months')

This troubleshooting guide covers the most common issues you'll encounter with the Boat Listing Platform. Keep it updated as you discover new issues and solutions.