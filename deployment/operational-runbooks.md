# Operational Runbooks

## Overview

This document provides comprehensive operational runbooks for the MarineMarket platform. These runbooks contain step-by-step procedures for common operational tasks, incident response, troubleshooting, and maintenance activities.

## Runbook Categories

### 1. Daily Operations
- [System Health Checks](#system-health-checks)
- [Performance Monitoring](#performance-monitoring)
- [Security Monitoring](#security-monitoring)
- [Backup Verification](#backup-verification)

### 2. Incident Response
- [Service Outage Response](#service-outage-response)
- [Performance Degradation](#performance-degradation)
- [Security Incident Response](#security-incident-response)
- [Data Loss Recovery](#data-loss-recovery)

### 3. Maintenance Procedures
- [Database Backup and Restore](#database-backup-and-restore)
- [Resource Cleanup](#resource-cleanup)
- [Performance Testing](#performance-testing)
- [Security Scanning](#security-scanning)

### 4. Troubleshooting Guides
- [Lambda Function Issues](#lambda-function-issues)
- [DynamoDB Performance Issues](#dynamodb-performance-issues)
- [API Gateway Problems](#api-gateway-problems)
- [Cloudflare and CDN Issues](#cloudflare-and-cdn-issues)

## Daily Operations

### System Health Checks

**Frequency**: Daily (automated with manual verification)
**Duration**: 15-30 minutes
**Prerequisites**: AWS CLI access, monitoring dashboard access

#### Procedure

1. **Check CloudWatch Dashboard**
   ```bash
   # Access admin service dashboard
   aws cloudwatch get-dashboard \
     --dashboard-name "admin-service-${ENVIRONMENT}" \
     --region us-east-1
   ```

2. **Verify All Services Are Running**
   ```bash
   # Check Lambda function status
   aws lambda list-functions \
     --query "Functions[?contains(FunctionName, 'BoatListingStack')].{Name:FunctionName,State:State}" \
     --region us-east-1
   
   # Check API Gateway health
   curl -f https://api.harborlist.com/health
   ```

3. **Review CloudWatch Alarms**
   ```bash
   # List active alarms
   aws cloudwatch describe-alarms \
     --state-value ALARM \
     --region us-east-1
   ```

4. **Check DynamoDB Table Status**
   ```bash
   # List all tables and their status
   aws dynamodb list-tables --region us-east-1
   
   # Check specific table status
   aws dynamodb describe-table \
     --table-name boat-listings \
     --region us-east-1 \
     --query "Table.TableStatus"
   ```

#### Success Criteria
- All Lambda functions in "Active" state
- API health endpoint returns HTTP 200
- No active CloudWatch alarms
- All DynamoDB tables in "ACTIVE" status

#### Escalation
If any checks fail, follow the [Service Outage Response](#service-outage-response) procedure.

### Performance Monitoring

**Frequency**: Daily
**Duration**: 10-15 minutes
**Prerequisites**: Performance testing tools, monitoring access

#### Procedure

1. **Run Basic Performance Test**
   ```bash
   # Execute performance test script
   ./scripts/operations/performance-test.sh ${ENVIRONMENT} --type basic --verbose
   ```

2. **Review Key Metrics**
   - Lambda function duration (should be < 1000ms average)
   - API response times (should be < 500ms for 95th percentile)
   - DynamoDB consumed capacity (should be < 80% of provisioned)
   - Error rates (should be < 1%)

3. **Check Resource Utilization**
   ```bash
   # Get Lambda function metrics
   aws cloudwatch get-metric-statistics \
     --namespace AWS/Lambda \
     --metric-name Duration \
     --dimensions Name=FunctionName,Value=BoatListingStack-${ENVIRONMENT}-AdminFunction \
     --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%SZ) \
     --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
     --period 300 \
     --statistics Average,Maximum \
     --region us-east-1
   ```

#### Success Criteria
- Performance test passes with < 5% error rate
- Response times within acceptable thresholds
- No resource utilization warnings

#### Escalation
If performance issues are detected, follow the [Performance Degradation](#performance-degradation) procedure.

### Security Monitoring

**Frequency**: Daily
**Duration**: 20-30 minutes
**Prerequisites**: Security scanning tools, audit log access

#### Procedure

1. **Run Security Scan**
   ```bash
   # Execute security scan script
   ./scripts/operations/security-scan.sh ${ENVIRONMENT} --type basic --verbose
   ```

2. **Review Audit Logs**
   ```bash
   # Check recent admin activities
   aws dynamodb scan \
     --table-name boat-audit-logs \
     --filter-expression "#ts > :yesterday" \
     --expression-attribute-names '{"#ts": "timestamp"}' \
     --expression-attribute-values '{":yesterday": {"S": "'$(date -u -d '1 day ago' +%Y-%m-%dT%H:%M:%SZ)'"}}' \
     --region us-east-1
   ```

3. **Check Failed Login Attempts**
   ```bash
   # Review login attempts from last 24 hours
   aws dynamodb scan \
     --table-name boat-login-attempts \
     --filter-expression "#ts > :yesterday AND #success = :false" \
     --expression-attribute-names '{"#ts": "timestamp", "#success": "success"}' \
     --expression-attribute-values '{":yesterday": {"S": "'$(date -u -d '1 day ago' +%Y-%m-%dT%H:%M:%SZ)'"}, ":false": {"BOOL": false}}' \
     --region us-east-1
   ```

4. **Verify SSL/TLS Configuration**
   ```bash
   # Check SSL certificate status
   curl -I https://api.harborlist.com
   
   # Verify SSL configuration
   openssl s_client -connect api.harborlist.com:443 -servername api.harborlist.com < /dev/null
   ```

#### Success Criteria
- Security scan passes with no critical issues
- No suspicious login patterns
- SSL/TLS certificates valid and properly configured
- No unauthorized access attempts

#### Escalation
If security issues are detected, follow the [Security Incident Response](#security-incident-response) procedure.

### Backup Verification

**Frequency**: Daily
**Duration**: 10-15 minutes
**Prerequisites**: AWS CLI access, backup scripts

#### Procedure

1. **Verify Recent Backups**
   ```bash
   # Check DynamoDB backups from last 24 hours
   aws dynamodb list-backups \
     --table-name boat-listings \
     --time-range-lower-bound $(date -u -d '1 day ago' +%Y-%m-%dT%H:%M:%SZ) \
     --region us-east-1
   ```

2. **Test Backup Integrity**
   ```bash
   # Run backup verification script
   ./scripts/operations/backup-database.sh ${ENVIRONMENT} --dry-run --verbose
   ```

3. **Check Backup Retention**
   ```bash
   # List all backups and check retention compliance
   aws dynamodb list-backups \
     --table-name boat-listings \
     --region us-east-1 \
     --query "BackupSummaries[?BackupType=='USER']"
   ```

#### Success Criteria
- Recent backups exist for all critical tables
- Backup integrity checks pass
- Backup retention policy is being followed

#### Escalation
If backup issues are detected, immediately create manual backups and investigate the automated backup process.

## Incident Response

### Service Outage Response

**Severity**: P1 (Critical)
**Response Time**: 15 minutes
**Prerequisites**: AWS CLI access, monitoring tools, communication channels

#### Immediate Response (0-15 minutes)

1. **Acknowledge the Incident**
   ```bash
   # Check overall system status
   curl -f https://api.harborlist.com/health || echo "API DOWN"
   curl -f https://harborlist.com || echo "FRONTEND DOWN"
   ```

2. **Identify Affected Components**
   ```bash
   # Check Lambda function status
   aws lambda list-functions \
     --query "Functions[?contains(FunctionName, 'BoatListingStack')].{Name:FunctionName,State:State,LastModified:LastModified}" \
     --region us-east-1
   
   # Check active alarms
   aws cloudwatch describe-alarms \
     --state-value ALARM \
     --region us-east-1
   ```

3. **Check Recent Deployments**
   ```bash
   # Check CloudFormation stack events
   aws cloudformation describe-stack-events \
     --stack-name BoatListingStack-${ENVIRONMENT} \
     --region us-east-1 \
     --query "StackEvents[0:10]"
   ```

#### Investigation Phase (15-60 minutes)

4. **Analyze Logs**
   ```bash
   # Check Lambda function logs
   aws logs filter-log-events \
     --log-group-name "/aws/lambda/BoatListingStack-${ENVIRONMENT}-AdminFunction" \
     --start-time $(date -d '1 hour ago' +%s)000 \
     --filter-pattern "ERROR" \
     --region us-east-1
   ```

5. **Check Dependencies**
   ```bash
   # Verify DynamoDB table accessibility
   aws dynamodb describe-table \
     --table-name boat-listings \
     --region us-east-1
   
   # Check API Gateway status
   aws apigateway get-rest-apis --region us-east-1
   ```

#### Resolution Actions

6. **Common Resolution Steps**
   
   **Lambda Function Issues:**
   ```bash
   # Restart Lambda function (update environment variable)
   aws lambda update-function-configuration \
     --function-name BoatListingStack-${ENVIRONMENT}-AdminFunction \
     --environment Variables="{RESTART_TIMESTAMP=$(date +%s)}" \
     --region us-east-1
   ```
   
   **DynamoDB Issues:**
   ```bash
   # Check table status and metrics
   aws dynamodb describe-table \
     --table-name boat-listings \
     --region us-east-1
   ```
   
   **API Gateway Issues:**
   ```bash
   # Create new deployment
   aws apigateway create-deployment \
     --rest-api-id $(aws apigateway get-rest-apis --query "items[?name=='Boat Listing API'].id" --output text) \
     --stage-name prod \
     --region us-east-1
   ```

#### Recovery Verification

7. **Verify Service Recovery**
   ```bash
   # Test all critical endpoints
   curl -f https://api.harborlist.com/health
   curl -f https://api.harborlist.com/listings
   curl -f https://harborlist.com
   ```

8. **Run Performance Test**
   ```bash
   ./scripts/operations/performance-test.sh ${ENVIRONMENT} --type basic
   ```

#### Post-Incident Actions

9. **Document the Incident**
   - Root cause analysis
   - Timeline of events
   - Resolution steps taken
   - Lessons learned
   - Prevention measures

### Performance Degradation

**Severity**: P2 (High)
**Response Time**: 1 hour
**Prerequisites**: Performance monitoring tools, AWS CLI access

#### Investigation Steps

1. **Identify Performance Bottlenecks**
   ```bash
   # Run comprehensive performance test
   ./scripts/operations/performance-test.sh ${ENVIRONMENT} --type load --duration 300 --users 20
   ```

2. **Analyze Lambda Performance**
   ```bash
   # Get detailed Lambda metrics
   aws cloudwatch get-metric-statistics \
     --namespace AWS/Lambda \
     --metric-name Duration \
     --dimensions Name=FunctionName,Value=BoatListingStack-${ENVIRONMENT}-AdminFunction \
     --start-time $(date -u -d '2 hours ago' +%Y-%m-%dT%H:%M:%SZ) \
     --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
     --period 300 \
     --statistics Average,Maximum,Minimum \
     --region us-east-1
   ```

3. **Check DynamoDB Performance**
   ```bash
   # Check consumed capacity and throttling
   aws cloudwatch get-metric-statistics \
     --namespace AWS/DynamoDB \
     --metric-name ConsumedReadCapacityUnits \
     --dimensions Name=TableName,Value=boat-listings \
     --start-time $(date -u -d '2 hours ago' +%Y-%m-%dT%H:%M:%SZ) \
     --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
     --period 300 \
     --statistics Sum \
     --region us-east-1
   ```

#### Resolution Actions

4. **Lambda Optimization**
   ```bash
   # Increase memory allocation if needed
   aws lambda update-function-configuration \
     --function-name BoatListingStack-${ENVIRONMENT}-AdminFunction \
     --memory-size 1024 \
     --region us-east-1
   ```

5. **DynamoDB Optimization**
   ```bash
   # Check for hot partitions and optimize queries
   # Review GSI usage and query patterns
   aws dynamodb describe-table \
     --table-name boat-listings \
     --region us-east-1 \
     --query "Table.GlobalSecondaryIndexes"
   ```

### Security Incident Response

**Severity**: P1 (Critical)
**Response Time**: 15 minutes
**Prerequisites**: Security tools, incident response team, communication channels

#### Immediate Response

1. **Assess the Threat**
   ```bash
   # Run immediate security scan
   ./scripts/operations/security-scan.sh ${ENVIRONMENT} --type comprehensive
   ```

2. **Check for Unauthorized Access**
   ```bash
   # Review recent login attempts
   aws dynamodb scan \
     --table-name boat-login-attempts \
     --filter-expression "#ts > :recent AND #success = :false" \
     --expression-attribute-names '{"#ts": "timestamp", "#success": "success"}' \
     --expression-attribute-values '{":recent": {"S": "'$(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%SZ)'"}, ":false": {"BOOL": false}}' \
     --region us-east-1
   ```

3. **Review Audit Logs**
   ```bash
   # Check for suspicious activities
   aws dynamodb scan \
     --table-name boat-audit-logs \
     --filter-expression "#ts > :recent" \
     --expression-attribute-names '{"#ts": "timestamp"}' \
     --expression-attribute-values '{":recent": {"S": "'$(date -u -d '2 hours ago' +%Y-%m-%dT%H:%M:%SZ)'"}}' \
     --region us-east-1
   ```

#### Containment Actions

4. **Isolate Affected Systems**
   ```bash
   # Disable compromised user accounts if identified
   # Update security groups to restrict access
   # Rotate API keys and secrets
   ```

5. **Preserve Evidence**
   ```bash
   # Create snapshots of affected resources
   # Export relevant logs
   # Document timeline of events
   ```

#### Recovery Actions

6. **Implement Security Fixes**
   ```bash
   # Update dependencies with security patches
   npm audit fix
   
   # Rotate secrets
   aws secretsmanager update-secret \
     --secret-id boat-listing-admin-jwt-${ENVIRONMENT} \
     --generate-secret-string \
     --region us-east-1
   ```

7. **Verify System Integrity**
   ```bash
   # Run comprehensive security scan
   ./scripts/operations/security-scan.sh ${ENVIRONMENT} --type comprehensive
   ```

## Maintenance Procedures

### Database Backup and Restore

**Frequency**: Daily (automated), On-demand (manual)
**Duration**: 30-60 minutes
**Prerequisites**: AWS CLI access, backup scripts

#### Creating Backups

The backup procedure is implemented in [`scripts/operations/backup-database.sh`](../../scripts/operations/backup-database.sh):

```bash
# Create full backup for production
./scripts/operations/backup-database.sh production --type full --verbose

# Create incremental backup for development
./scripts/operations/backup-database.sh dev --type incremental --retention 7
```

#### Backup Script Features

- **Automatic Discovery**: Finds DynamoDB tables for the specified environment
- **Retention Management**: Automatically cleans up old backups based on retention policy
- **Metadata Tracking**: Creates detailed backup metadata for recovery purposes
- **Multiple Environments**: Supports dev, staging, and production environments

#### Restoring from Backup

1. **List Available Backups**
   ```bash
   # List backups for a specific table
   aws dynamodb list-backups \
     --table-name boat-listings \
     --region us-east-1
   ```

2. **Restore Table from Backup**
   ```bash
   # Restore to new table
   aws dynamodb restore-table-from-backup \
     --target-table-name boat-listings-restored \
     --backup-arn arn:aws:dynamodb:us-east-1:account:table/boat-listings/backup/backup-id \
     --region us-east-1
   ```

3. **Verify Restored Data**
   ```bash
   # Check item count
   aws dynamodb describe-table \
     --table-name boat-listings-restored \
     --region us-east-1 \
     --query "Table.ItemCount"
   ```

### Resource Cleanup

**Frequency**: Weekly
**Duration**: 45-90 minutes
**Prerequisites**: AWS CLI access, cleanup scripts

The cleanup procedure is implemented in [`scripts/operations/cleanup-resources.sh`](../../scripts/operations/cleanup-resources.sh):

#### Safe Cleanup (Recommended)
```bash
# Preview cleanup actions
./scripts/operations/cleanup-resources.sh ${ENVIRONMENT} --type safe --dry-run --verbose

# Execute safe cleanup
./scripts/operations/cleanup-resources.sh ${ENVIRONMENT} --type safe --verbose
```

#### Cleanup Types

- **Safe**: Removes old logs (>30 days), unused snapshots (>30 days)
- **Moderate**: Safe + old AMIs (>7 days), unattached volumes (>7 days)
- **Aggressive**: Moderate + old backups (>1 day), temporary resources

#### Cleanup Features

- **Safety Confirmations**: Requires explicit confirmation for destructive actions
- **Dry Run Mode**: Preview changes before execution
- **Environment Filtering**: Only cleans resources matching the environment
- **Detailed Reporting**: Provides summary of cleaned resources

### Performance Testing

**Frequency**: Weekly, Before major releases
**Duration**: 30-60 minutes
**Prerequisites**: Performance testing tools

The performance testing procedure is implemented in [`scripts/operations/performance-test.sh`](../../scripts/operations/performance-test.sh):

#### Basic Performance Test
```bash
# Run basic functionality and response time tests
./scripts/operations/performance-test.sh ${ENVIRONMENT} --type basic --verbose
```

#### Load Testing
```bash
# Run sustained load test
./scripts/operations/performance-test.sh ${ENVIRONMENT} --type load --users 50 --duration 600
```

#### Stress Testing
```bash
# Run high load test to find breaking points
./scripts/operations/performance-test.sh ${ENVIRONMENT} --type stress --users 100 --duration 300
```

#### Performance Test Features

- **Multiple Test Types**: Basic, load, stress, and spike testing
- **Tool Integration**: Supports wrk, hey, Apache Bench, and basic curl testing
- **Automatic URL Discovery**: Finds API endpoints from CloudFormation outputs
- **Detailed Reporting**: Generates comprehensive performance reports

### Security Scanning

**Frequency**: Daily (basic), Weekly (comprehensive)
**Duration**: 20-45 minutes
**Prerequisites**: Security scanning tools

The security scanning procedure is implemented in [`scripts/operations/security-scan.sh`](../../scripts/operations/security-scan.sh):

#### Basic Security Scan
```bash
# Run basic dependency and configuration checks
./scripts/operations/security-scan.sh ${ENVIRONMENT} --type basic --verbose
```

#### Comprehensive Security Scan
```bash
# Run full security audit including code analysis
./scripts/operations/security-scan.sh ${ENVIRONMENT} --type comprehensive --format json
```

#### Infrastructure Security Scan
```bash
# Run infrastructure and AWS configuration security checks
./scripts/operations/security-scan.sh ${ENVIRONMENT} --type infrastructure --verbose
```

#### Security Scan Features

- **Dependency Scanning**: Uses npm audit for vulnerability detection
- **Secret Detection**: Scans for exposed secrets and credentials
- **Infrastructure Security**: Validates AWS resource configurations
- **Multiple Output Formats**: JSON, text, and SARIF formats
- **Risk Assessment**: Provides overall risk level and recommendations

## Troubleshooting Guides

### Lambda Function Issues

#### Common Symptoms
- Function timeouts
- High error rates
- Cold start performance issues
- Memory or CPU limitations

#### Diagnostic Steps

1. **Check Function Status**
   ```bash
   aws lambda get-function \
     --function-name BoatListingStack-${ENVIRONMENT}-AdminFunction \
     --region us-east-1
   ```

2. **Review Function Logs**
   ```bash
   aws logs filter-log-events \
     --log-group-name "/aws/lambda/BoatListingStack-${ENVIRONMENT}-AdminFunction" \
     --start-time $(date -d '1 hour ago' +%s)000 \
     --region us-east-1
   ```

3. **Check Function Metrics**
   ```bash
   aws cloudwatch get-metric-statistics \
     --namespace AWS/Lambda \
     --metric-name Errors \
     --dimensions Name=FunctionName,Value=BoatListingStack-${ENVIRONMENT}-AdminFunction \
     --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%SZ) \
     --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
     --period 300 \
     --statistics Sum \
     --region us-east-1
   ```

#### Resolution Actions

**Timeout Issues:**
```bash
# Increase timeout
aws lambda update-function-configuration \
  --function-name BoatListingStack-${ENVIRONMENT}-AdminFunction \
  --timeout 60 \
  --region us-east-1
```

**Memory Issues:**
```bash
# Increase memory allocation
aws lambda update-function-configuration \
  --function-name BoatListingStack-${ENVIRONMENT}-AdminFunction \
  --memory-size 1024 \
  --region us-east-1
```

**Cold Start Issues:**
```bash
# Enable provisioned concurrency
aws lambda put-provisioned-concurrency-config \
  --function-name BoatListingStack-${ENVIRONMENT}-AdminFunction \
  --qualifier '$LATEST' \
  --provisioned-concurrency-units 2 \
  --region us-east-1
```

### DynamoDB Performance Issues

#### Common Symptoms
- High latency
- Throttled requests
- Hot partition issues
- Capacity exceeded errors

#### Diagnostic Steps

1. **Check Table Metrics**
   ```bash
   aws cloudwatch get-metric-statistics \
     --namespace AWS/DynamoDB \
     --metric-name ThrottledRequests \
     --dimensions Name=TableName,Value=boat-listings \
     --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%SZ) \
     --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
     --period 300 \
     --statistics Sum \
     --region us-east-1
   ```

2. **Analyze Access Patterns**
   ```bash
   # Check consumed capacity
   aws cloudwatch get-metric-statistics \
     --namespace AWS/DynamoDB \
     --metric-name ConsumedReadCapacityUnits \
     --dimensions Name=TableName,Value=boat-listings \
     --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%SZ) \
     --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
     --period 300 \
     --statistics Sum \
     --region us-east-1
   ```

#### Resolution Actions

**Throttling Issues:**
- Review query patterns for hot partitions
- Implement exponential backoff in application code
- Consider using Global Secondary Indexes for different access patterns

**Capacity Issues:**
- Monitor auto-scaling settings
- Adjust read/write capacity units if using provisioned mode
- Consider switching to on-demand billing mode

### API Gateway Problems

#### Common Symptoms
- High latency
- 5xx errors
- Rate limiting issues
- CORS problems

#### Diagnostic Steps

1. **Check API Gateway Logs**
   ```bash
   # Enable CloudWatch logs for API Gateway if not already enabled
   aws logs filter-log-events \
     --log-group-name "API-Gateway-Execution-Logs_${API_ID}/prod" \
     --start-time $(date -d '1 hour ago' +%s)000 \
     --region us-east-1
   ```

2. **Test API Endpoints**
   ```bash
   # Test with verbose output
   curl -v https://api.harborlist.com/health
   curl -v https://api.harborlist.com/listings
   ```

#### Resolution Actions

**CORS Issues:**
```bash
# Update CORS configuration in CDK and redeploy
# Check preflight OPTIONS requests
curl -X OPTIONS -H "Origin: https://harborlist.com" \
  -H "Access-Control-Request-Method: POST" \
  -H "Access-Control-Request-Headers: Content-Type" \
  https://api.harborlist.com/listings
```

**Rate Limiting:**
- Review usage plans and API keys
- Implement proper rate limiting in application
- Monitor throttling metrics

### Cloudflare and CDN Issues

#### Common Symptoms
- Slow content delivery
- Cache misses
- Origin access errors
- SSL/TLS issues
- Tunnel daemon connectivity problems

#### Diagnostic Steps

1. **Check Cloudflare Tunnel Status**
   ```bash
   # Check tunnel daemon status on EC2
   sudo systemctl status cloudflared
   # View tunnel logs
   sudo journalctl -u cloudflared -f
   ```

2. **Verify VPC Endpoint**
   ```bash
   # Check VPC endpoint status
   aws ec2 describe-vpc-endpoints --filters Name=service-name,Values=com.amazonaws.us-east-1.s3
   # Test S3 access via VPC endpoint
   curl -I http://bucket-name.s3-website-us-east-1.amazonaws.com/
   ```

3. **Test Cache Behavior**
   ```bash
   # Check cache headers via Cloudflare
   curl -I https://harborlist.com
   curl -I https://harborlist.com/static/js/main.js
   ```

#### Resolution Actions

**Cache Issues:**
- Review Cloudflare cache rules and page rules
- Implement proper cache headers in S3 website hosting
- Use Cloudflare cache purge when needed

**Tunnel Issues:**
- Restart cloudflared daemon: `sudo systemctl restart cloudflared`
- Verify tunnel configuration and credentials
- Check EC2 instance health and security groups

**VPC Endpoint Issues:**
- Verify VPC endpoint route table configuration
- Check S3 bucket policy allows VPC endpoint access
- Review security group rules for EC2 tunnel daemon

## Emergency Contacts and Escalation

### Contact Information

**Primary On-Call Engineer**
- Name: [Primary Engineer Name]
- Phone: [Phone Number]
- Email: [Email Address]
- Slack: @[username]

**Secondary On-Call Engineer**
- Name: [Secondary Engineer Name]
- Phone: [Phone Number]
- Email: [Email Address]
- Slack: @[username]

**Team Lead**
- Name: [Team Lead Name]
- Phone: [Phone Number]
- Email: [Email Address]
- Slack: @[username]

### Escalation Matrix

| Severity | Response Time | Escalation Path |
|----------|---------------|-----------------|
| P1 (Critical) | 15 minutes | Primary → Secondary → Team Lead → Management |
| P2 (High) | 1 hour | Primary → Team Lead |
| P3 (Medium) | 4 hours | Primary → Team Lead (next business day) |
| P4 (Low) | Next business day | Standard ticket process |

### Communication Channels

**Slack Channels:**
- `#incidents` - Active incident coordination
- `#alerts` - Automated monitoring alerts
- `#deployments` - Deployment notifications
- `#general` - General team communication

**Email Lists:**
- `team-oncall@company.com` - On-call team notifications
- `team-leads@company.com` - Management escalation
- `all-engineering@company.com` - Major incident notifications

## Documentation and References

### Related Documentation
- [Infrastructure Setup Guide](infrastructure-setup.md)
- [Monitoring and Alerting Guide](monitoring.md)
- [Cost Optimization Guide](cost-optimization.md)
- [Security Architecture](../security/)
- [Performance Testing Guide](testing/performance-testing.md)

### External Resources
- [AWS Lambda Troubleshooting](https://docs.aws.amazon.com/lambda/latest/dg/troubleshooting.html)
- [DynamoDB Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html)
- [API Gateway Troubleshooting](https://docs.aws.amazon.com/apigateway/latest/developerguide/troubleshooting.html)
- [CloudWatch User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/)

### Runbook Maintenance

**Review Schedule**: Monthly
**Update Process**: 
1. Review runbook effectiveness after incidents
2. Update procedures based on lessons learned
3. Test procedures in non-production environments
4. Get team review and approval for changes

**Version Control**: All runbooks are version controlled in the project repository under `docs/deployment/operational-runbooks.md`

---

**Last Updated**: [Current Date]
**Version**: 1.0
**Reviewed By**: [Team Lead Name]
**Next Review Date**: [Date + 1 month]