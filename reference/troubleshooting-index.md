# Troubleshooting Index

## Overview

This comprehensive troubleshooting index provides quick access to solutions for common issues across the MarineMarket platform. Issues are organized by category with cross-references to detailed documentation.

## 🚨 Emergency Quick Reference

### Critical System Failures

| Issue | Quick Fix | Detailed Guide |
|-------|-----------|----------------|
| **Complete site down** | `curl -I https://harborlist.com` → Check AWS status → Restart services | [Emergency Recovery](#emergency-recovery) |
| **Database unavailable** | Check DynamoDB status → Verify IAM permissions → Create backup | [Database Issues](#database-issues) |
| **API completely failing** | Test endpoints → Check Lambda logs → Verify API Gateway | [API Issues](#api-issues) |
| **Admin panel inaccessible** | Verify admin user exists → Check authentication → Reset credentials | [Admin Issues](#admin-issues) |

### Essential Diagnostic Commands

```bash
# System Health Check
curl -I https://harborlist.com && curl -I https://api.harborlist.com/health

# AWS Infrastructure Status
aws cloudformation describe-stacks --stack-name BoatListingStack-prod --query 'Stacks[0].StackStatus'

# Service Logs
aws logs filter-log-events --log-group-name "/aws/lambda/BoatListingStack-ListingFunction" --start-time $(date -d '1 hour ago' +%s)000

# Database Connectivity
aws dynamodb describe-table --table-name boat-users-prod
```

## 📱 Frontend Issues

### Development Environment

#### Build and Compilation Issues

**TypeScript Compilation Errors**
- **Symptoms:** Build fails with TS errors, IDE shows type errors
- **Quick Fix:** 
  ```bash
  rm -rf node_modules/.cache && npm run type-check
  ```
- **Common Causes:**
  - Outdated dependencies
  - Missing type definitions
  - Configuration conflicts
- **Detailed Solutions:**
  - Clear TypeScript cache: `rm -rf .tsbuildinfo`
  - Reinstall dependencies: `rm -rf node_modules package-lock.json && npm install`
  - Update type definitions: `npm update @types/*`
- **Related:** [Development Setup](development/local-setup.md), [Code Structure](development/code-structure.md)

**Vite Build Failures**
- **Symptoms:** `npm run build` fails, missing modules, memory errors
- **Quick Fix:**
  ```bash
  npm run clean && npm install && npm run build
  ```
- **Common Causes:**
  - Insufficient memory
  - Circular dependencies
  - Missing environment variables
- **Detailed Solutions:**
  - Increase Node.js memory: `NODE_OPTIONS="--max-old-space-size=4096" npm run build`
  - Check for circular imports: Use dependency analysis tools
  - Verify environment variables: Check `.env.local` configuration
- **Related:** [Build Configuration](development/development-workflow.md)

#### Development Server Issues

**Port Already in Use (EADDRINUSE)**
- **Symptoms:** `npm run dev` fails, port 5173 unavailable
- **Quick Fix:**
  ```bash
  lsof -ti:5173 | xargs kill -9 && npm run dev
  ```
- **Alternative Solutions:**
  - Use different port: `npm run dev -- --port 3001`
  - Check for zombie processes: `ps aux | grep node`
- **Prevention:** Always stop dev server properly with Ctrl+C

**Hot Module Replacement Not Working**
- **Symptoms:** Changes not reflected, manual refresh required
- **Quick Fix:** Restart dev server and clear browser cache
- **Common Causes:**
  - File watcher limits exceeded
  - Browser cache issues
  - Proxy configuration problems
- **Solutions:**
  - Increase file watchers: `echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf`
  - Clear browser cache and storage
  - Check Vite configuration for proxy settings

#### API Integration Issues

**CORS Errors**
- **Symptoms:** Browser console shows CORS errors, API requests blocked
- **Quick Fix:** Verify API URL in environment variables
- **Common Causes:**
  - Incorrect API URL configuration
  - Missing CORS headers on backend
  - Development vs production URL mismatch
- **Solutions:**
  ```bash
  # Check environment configuration
  echo $VITE_API_BASE_URL
  
  # Test API directly
  curl -v https://api.harborlist.com/health
  
  # Test CORS preflight
  curl -X OPTIONS -H "Origin: https://harborlist.com" https://api.harborlist.com/listings
  ```
- **Related:** [API Reference](../development/api-reference.md), [Environment Configuration](deployment/infrastructure-setup.md)

**Authentication Token Issues**
- **Symptoms:** Login fails, token expired errors, 401 responses
- **Quick Fix:**
  ```javascript
  // Clear token and re-authenticate
  localStorage.removeItem('authToken');
  window.location.reload();
  ```
- **Debugging Steps:**
  ```javascript
  // Check token in browser
  const token = localStorage.getItem('authToken');
  console.log('Token:', token);
  
  // Decode JWT payload
  if (token) {
    const payload = JSON.parse(atob(token.split('.')[1]));
    console.log('Expires:', new Date(payload.exp * 1000));
  }
  ```
- **Related:** [Authentication Guide](security/authentication.md)

### Production Issues

**Static Asset Loading Failures**
- **Symptoms:** CSS/JS files not loading, 404 errors for assets
- **Quick Fix:** Check Cloudflare configuration and S3 website hosting
- **Common Causes:**
  - Incorrect asset paths
  - Cloudflare cache issues
  - S3 bucket policy or VPC endpoint problems
  - Cloudflare Tunnel daemon issues
- **Solutions:**
  ```bash
  # Check S3 bucket contents
  aws s3 ls s3://boat-listing-frontend-ACCOUNT/
  # Test S3 website endpoint via VPC
  curl -I http://bucket-name.s3-website-us-east-1.amazonaws.com/
  # Check VPC endpoint status
  aws ec2 describe-vpc-endpoints --filters Name=service-name,Values=com.amazonaws.us-east-1.s3
  # Purge Cloudflare cache
  # Use Cloudflare dashboard or API to purge cache
  ```

**Performance Issues**
- **Symptoms:** Slow loading times, poor Core Web Vitals scores
- **Quick Diagnosis:**
  ```bash
  # Test response times
  curl -w "@curl-format.txt" -o /dev/null -s https://harborlist.com
  ```
- **Common Solutions:**
  - Enable Cloudflare optimization features (Auto Minify, Rocket Loader, etc.)
  - Verify VPC endpoint is functioning correctly
  - Implement lazy loading for images
  - Optimize bundle size with code splitting
  - Use Cloudflare's automatic image optimization
- **Related:** [Performance Testing](testing/performance-testing.md)

## 🔧 Backend Issues

### Lambda Function Problems

**Function Deployment Failures**
- **Symptoms:** CDK deploy fails, Lambda update errors, package size issues
- **Quick Fix:**
  ```bash
  cd backend && npm run clean && npm run build
  ```
- **Common Causes:**
  - Package size exceeds 50MB limit
  - Missing dependencies
  - Handler configuration errors
- **Solutions:**
  ```bash
  # Check package sizes
  ls -lh backend/dist/packages/*.zip
  
  # Rebuild with production dependencies only
  cd backend && ./build.sh
  
  # Verify handler configuration
  grep -r "handler:" infrastructure/lib/
  ```
- **Related:** [Backend Architecture](architecture/backend-architecture.md)

**Runtime Errors**
- **Symptoms:** 500 errors, function timeouts, memory errors
- **Quick Diagnosis:**
  ```bash
  # Check CloudWatch logs
  aws logs filter-log-events \
    --log-group-name "/aws/lambda/BoatListingStack-ListingFunction" \
    --start-time $(date -d '1 hour ago' +%s)000
  ```
- **Common Issues:**
  - Memory limit exceeded
  - Timeout errors
  - Unhandled promise rejections
  - Cold start performance
- **Solutions:**
  - Increase memory allocation in CDK
  - Optimize function code for performance
  - Implement proper error handling
  - Use provisioned concurrency for critical functions

**Database Connection Issues**
- **Symptoms:** DynamoDB operations fail, permission denied, table not found
- **Quick Fix:**
  ```bash
  # Test table access
  aws dynamodb describe-table --table-name boat-users-prod
  ```
- **Common Causes:**
  - IAM permission issues
  - Incorrect table names
  - Region mismatch
  - VPC configuration problems
- **Solutions:**
  ```bash
  # Check IAM permissions
  aws iam get-role-policy --role-name BoatListingStack-ListingFunctionRole --policy-name DynamoDBAccess
  
  # Verify environment variables
  aws lambda get-function-configuration --function-name BoatListingStack-ListingFunction
  
  # Test DynamoDB access
  aws dynamodb scan --table-name boat-users-prod --limit 1
  ```

### API Gateway Issues

**CORS Configuration Problems**
- **Symptoms:** Preflight requests fail, CORS errors in browser
- **Quick Fix:** Verify CORS configuration in CDK
- **Testing:**
  ```bash
  # Test preflight request
  curl -X OPTIONS \
    -H "Origin: https://harborlist.com" \
    -H "Access-Control-Request-Method: GET" \
    -H "Access-Control-Request-Headers: Content-Type" \
    https://api.harborlist.com/listings
  ```

**Custom Domain Issues**
- **Symptoms:** API domain not resolving, SSL certificate errors
- **Quick Diagnosis:**
  ```bash
  # Check certificate in ACM
  aws acm list-certificates --region us-east-1
  
  # Test SSL configuration
  openssl s_client -connect api.harborlist.com:443 -servername api.harborlist.com
  ```
- **Related:** [Domain Setup](deployment/infrastructure-setup.md)

**Rate Limiting Issues**
- **Symptoms:** 429 Too Many Requests errors
- **Quick Fix:** Check API Gateway throttling settings
- **Solutions:**
  - Increase throttling limits
  - Implement client-side retry logic
  - Use exponential backoff
  - Monitor usage patterns

## 🗄️ Database Issues

### DynamoDB Problems

**Table Access Denied**
- **Symptoms:** 403 Forbidden errors, permission denied
- **Quick Fix:**
  ```bash
  # Check table exists and permissions
  aws dynamodb describe-table --table-name boat-users-prod
  ```
- **Common Causes:**
  - IAM policy issues
  - Resource-based policies
  - Cross-account access problems
- **Solutions:**
  - Verify IAM role has DynamoDB permissions
  - Check resource-based policies on tables
  - Ensure correct AWS region

**Performance Issues**
- **Symptoms:** High latency, throttling errors, consumed capacity warnings
- **Quick Diagnosis:**
  ```bash
  # Check table metrics
  aws cloudwatch get-metric-statistics \
    --namespace AWS/DynamoDB \
    --metric-name ConsumedReadCapacityUnits \
    --dimensions Name=TableName,Value=boat-listings-prod \
    --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
    --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
    --period 300 \
    --statistics Sum
  ```
- **Solutions:**
  - Optimize query patterns
  - Use Global Secondary Indexes (GSI) appropriately
  - Consider on-demand billing mode
  - Implement caching strategies

**Data Consistency Issues**
- **Symptoms:** Stale data, eventual consistency problems
- **Solutions:**
  - Use strongly consistent reads when necessary
  - Implement proper error handling for retries
  - Consider DynamoDB Streams for real-time updates

### Backup and Recovery

**Backup Failures**
- **Symptoms:** Backup creation fails, missing backups
- **Quick Fix:**
  ```bash
  # Create manual backup
  aws dynamodb create-backup \
    --table-name boat-users-prod \
    --backup-name emergency-backup-$(date +%Y%m%d-%H%M%S)
  ```
- **Related:** [Backup Procedures](operations/backup-recovery.md)

**Data Recovery Issues**
- **Symptoms:** Need to restore from backup, data corruption
- **Emergency Procedure:**
  ```bash
  # List available backups
  aws dynamodb list-backups --table-name boat-users-prod
  
  # Restore from backup
  aws dynamodb restore-table-from-backup \
    --target-table-name boat-users-prod-restored \
    --backup-arn BACKUP-ARN
  ```

## 🌐 Infrastructure Issues

### Cloudflare Tunnel Problems

**Tunnel Service Down**
- **Symptoms:** 502/503 errors, tunnel shows as disconnected
- **Quick Fix:**
  ```bash
  # Get EC2 instance and restart tunnel
  INSTANCE_ID=$(aws cloudformation describe-stacks \
    --stack-name BoatListingStack-dev \
    --query 'Stacks[0].Outputs[?OutputKey==`TunnelInstanceId`].OutputValue' \
    --output text)
  
  aws ssm start-session --target $INSTANCE_ID
  sudo systemctl restart cloudflared
  ```
- **Common Causes:**
  - Service crashed or stopped
  - Configuration file issues
  - Network connectivity problems
  - Resource exhaustion

**Configuration Issues**
- **Symptoms:** Tunnel won't start, routing errors, SSL problems
- **Diagnosis:**
  ```bash
  # Check configuration
  sudo cat /etc/cloudflared/config.yml
  
  # Validate YAML syntax
  python3 -c "import yaml; yaml.safe_load(open('/etc/cloudflared/config.yml'))"
  
  # Test manually
  sudo -u cloudflared cloudflared tunnel --config /etc/cloudflared/config.yml run [TUNNEL_ID]
  ```

**DNS Resolution Problems**
- **Symptoms:** Domain not resolving, wrong IP addresses
- **Quick Fix:**
  ```bash
  # Test DNS resolution
  nslookup harborlist.com
  dig harborlist.com
  
  # Test from different DNS servers
  nslookup harborlist.com 8.8.8.8
  ```
- **Solutions:**
  - Check Cloudflare DNS settings
  - Verify CNAME records point to tunnel
  - Clear local DNS cache
  - Wait for DNS propagation (up to 24 hours)

### AWS Infrastructure Issues

**CDK Deployment Failures**
- **Symptoms:** Stack creation fails, resource errors, rollback occurs
- **Quick Diagnosis:**
  ```bash
  # Check stack events
  aws cloudformation describe-stack-events --stack-name BoatListingStack-prod
  
  # View detailed errors
  aws cloudformation describe-stack-resources --stack-name BoatListingStack-prod
  ```
- **Common Solutions:**
  - Check AWS service limits
  - Verify IAM permissions
  - Ensure unique resource names
  - Check resource dependencies

**VPC and Networking Issues**
- **Symptoms:** Lambda functions can't access internet, VPC endpoint issues
- **Quick Fix:**
  ```bash
  # Check VPC configuration
  aws ec2 describe-vpcs
  aws ec2 describe-subnets
  aws ec2 describe-route-tables
  ```
- **Common Causes:**
  - Missing NAT Gateway
  - Incorrect route table configuration
  - Security group restrictions
  - VPC endpoint misconfiguration

**S3 Access Issues**
- **Symptoms:** 403 Forbidden errors, bucket access denied
- **Quick Diagnosis:**
  ```bash
  # Test S3 access
  aws s3 ls s3://boat-listing-frontend-ACCOUNT/
  
  # Check bucket policy
  aws s3api get-bucket-policy --bucket boat-listing-frontend-ACCOUNT
  ```
- **Solutions:**
  - Verify bucket policy allows required access
  - Check VPC endpoint configuration
  - Ensure correct IAM permissions
  - Verify bucket exists in correct region

## 🔐 Security Issues

### Authentication Problems

**JWT Token Issues**
- **Symptoms:** Authentication fails, token expired, invalid signature
- **Quick Diagnosis:**
  ```javascript
  // Check token in browser
  const token = localStorage.getItem('authToken');
  if (token) {
    const payload = JSON.parse(atob(token.split('.')[1]));
    console.log('Token expires:', new Date(payload.exp * 1000));
  }
  ```
- **Common Solutions:**
  - Implement token refresh mechanism
  - Check JWT secret configuration
  - Verify token expiration times
  - Handle token validation errors properly

**Admin Access Issues**
- **Symptoms:** Cannot access admin panel, permission denied
- **Quick Fix:**
  ```bash
  # Check if admin user exists
  aws dynamodb query \
    --table-name boat-users-prod \
    --index-name email-index \
    --key-condition-expression "email = :email" \
    --expression-attribute-values '{":email":{"S":"admin@example.com"}}'
  ```
- **Solutions:**
  ```bash
  # Create admin user
  cd backend
  npm run create-admin -- --email admin@example.com --name "Admin" --role super_admin
  
  # Reset admin password
  npm run create-admin -- --email admin@example.com --password "NewPassword123!"
  ```

**MFA Issues**
- **Symptoms:** MFA setup fails, invalid codes, authentication loops
- **Common Solutions:**
  - Verify time synchronization on devices
  - Check MFA secret generation
  - Implement backup codes
  - Provide clear setup instructions

### SSL/TLS Issues

**Certificate Problems**
- **Symptoms:** SSL warnings, certificate errors, HTTPS not working
- **Quick Diagnosis:**
  ```bash
  # Check certificate details
  openssl s_client -connect harborlist.com:443 -servername harborlist.com
  
  # Check ACM certificates
  aws acm list-certificates --region us-east-1
  ```
- **Solutions:**
  - Verify certificate is validated in ACM
  - Check Cloudflare SSL settings
  - Ensure certificate covers all required domains
  - Monitor certificate expiration dates

## 📊 Performance Issues

### Slow Response Times

**Frontend Performance**
- **Symptoms:** Slow page loads, poor Core Web Vitals
- **Quick Diagnosis:**
  ```bash
  # Test response times
  curl -w "@curl-format.txt" -o /dev/null -s https://harborlist.com
  ```
- **Solutions:**
  - Optimize bundle size with code splitting
  - Implement lazy loading for images
  - Use CDN for static assets
  - Enable compression and caching
  - Optimize images and media files

**Backend Performance**
- **Symptoms:** High API response times, Lambda timeouts
- **Quick Diagnosis:**
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
  ```
- **Solutions:**
  - Optimize database queries
  - Implement caching strategies
  - Increase Lambda memory allocation
  - Use connection pooling
  - Optimize cold start performance

### High Resource Usage

**Memory Issues**
- **Symptoms:** Out of memory errors, Lambda memory exceeded
- **Solutions:**
  - Increase Lambda memory allocation
  - Optimize memory usage in code
  - Implement streaming for large data
  - Use pagination for large datasets

**CPU Issues**
- **Symptoms:** High CPU utilization, slow processing
- **Solutions:**
  - Optimize algorithms and data structures
  - Use asynchronous processing
  - Implement caching
  - Consider using more powerful instance types

## 🔍 Monitoring and Debugging

### Log Analysis

**CloudWatch Logs**
- **Common Commands:**
  ```bash
  # View recent Lambda logs
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

**Application Debugging**
- **Frontend Debugging:**
  ```javascript
  // Enable debug mode
  localStorage.setItem('debug', 'true');
  
  // Check React Query cache
  import { useQueryClient } from '@tanstack/react-query';
  const queryClient = useQueryClient();
  console.log(queryClient.getQueryCache().getAll());
  ```
- **Backend Debugging:**
  ```typescript
  // Add debug logging
  console.log('Debug info:', { userId, requestId, data });
  
  // Use X-Ray for tracing
  import AWSXRay from 'aws-xray-sdk-core';
  const segment = AWSXRay.getSegment();
  segment.addAnnotation('userId', userId);
  ```

### Metrics and Alerting

**Key Metrics to Monitor:**
- HTTP response codes and error rates
- API response times and throughput
- Lambda function duration and errors
- DynamoDB consumed capacity and throttling
- CloudFront cache hit ratio
- EC2 instance CPU and memory usage

**Setting Up Alerts:**
```bash
# High error rate alert
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

## 🚨 Emergency Recovery

### Complete System Outage

**Immediate Response:**
1. **Check AWS Service Health**
   - Visit: https://status.aws.amazon.com/
   - Check us-east-1 region status

2. **Quick Recovery Steps:**
   ```bash
   # Restart critical services
   aws ec2 reboot-instances --instance-ids $INSTANCE_ID
   
   # Wait and test
   sleep 300
   curl -I https://harborlist.com
   ```

3. **Emergency DNS Failover:**
   - Point DNS to maintenance page
   - Update Cloudflare DNS records
   - Notify users via status page

### Database Recovery

**Immediate Backup:**
```bash
# Create emergency backup
aws dynamodb create-backup \
  --table-name boat-users-prod \
  --backup-name emergency-backup-$(date +%Y%m%d-%H%M%S)
```

**Recovery from Backup:**
```bash
# List available backups
aws dynamodb list-backups --table-name boat-users-prod

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
./scripts/deployment/deploy-production.sh --force
```

## 📞 Escalation Procedures

### Internal Escalation Path

1. **Level 1 - Self-Service**
   - Check this troubleshooting guide
   - Review system logs and metrics
   - Try common solutions

2. **Level 2 - Team Support**
   - Contact development team
   - Provide detailed error information
   - Include steps already tried

3. **Level 3 - Infrastructure Team**
   - Escalate to DevOps/Infrastructure team
   - Include system-wide impact assessment
   - Provide timeline and business impact

4. **Level 4 - External Support**
   - Contact AWS Support (if applicable)
   - Engage Cloudflare Support
   - Consider emergency vendor support

### Support Ticket Information

**Always Include:**
- Environment (dev/staging/prod)
- Exact error messages and logs
- Steps to reproduce the issue
- Timeline of when issue started
- Impact on users and business
- Troubleshooting steps already attempted
- System metrics and monitoring data

### External Support Resources

- **AWS Support**: Available through AWS Console
- **Cloudflare Support**: Available through Cloudflare Dashboard  
- **Community Forums**: Stack Overflow, AWS Forums, Reddit
- **Documentation**: AWS Docs, Cloudflare Docs, React Docs

## 🛡️ Prevention and Best Practices

### Proactive Monitoring

**Health Checks:**
```bash
#!/bin/bash
# health-check.sh - Run every 5 minutes

DOMAIN="harborlist.com"
API_DOMAIN="api.harborlist.com"

# Test website
if ! curl -f -s https://$DOMAIN > /dev/null; then
    echo "ALERT: Website is down"
    # Send notification
fi

# Test API
if ! curl -f -s https://$API_DOMAIN/health > /dev/null; then
    echo "ALERT: API is down"
    # Send notification
fi
```

**Regular Maintenance:**
- **Daily:** Monitor dashboards and alerts
- **Weekly:** Review performance metrics and logs
- **Monthly:** Update dependencies and security patches
- **Quarterly:** Disaster recovery testing

### Documentation Updates

**After Resolving Issues:**
1. Document the problem and solution
2. Update this troubleshooting guide
3. Share learnings with the team
4. Update monitoring and alerting
5. Create preventive measures
6. Review and update runbooks

### Continuous Improvement

**Post-Incident Reviews:**
- Conduct blameless post-mortems
- Identify root causes
- Implement preventive measures
- Update documentation and procedures
- Share learnings across teams

## 📚 Related Documentation

### Detailed Guides
- [Development Setup](development/local-setup.md)
- [Deployment Guide](deployment/infrastructure-setup.md)
- [Security Overview](security/security-overview.md)
- [Performance Testing](testing/performance-testing.md)
- [Monitoring Guide](operations/monitoring.md)

### Reference Materials
- [API Reference](../development/api-reference.md)
- [CLI Commands](cli-commands.md)
- [Configuration Reference](../reference/configuration.md)
- [Architecture Overview](architecture/overview.md)

### Operational Procedures
- [Backup and Recovery](operations/backup-recovery.md)
- [Incident Response](operations/incident-response.md)
- [Maintenance Procedures](operations/maintenance.md)

---

**Last Updated:** December 2024  
**Version:** 1.0  
**Next Review:** March 2025

This troubleshooting index is a living document. Please contribute by adding new issues and solutions as they are discovered and resolved.