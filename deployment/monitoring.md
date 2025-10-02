# Monitoring and Alerting Guide

## Overview

This guide covers the comprehensive monitoring and alerting setup for the MarineMarket platform. The monitoring infrastructure is implemented using AWS CloudWatch, SNS notifications, and custom dashboards to ensure system reliability, performance, and security.

## Monitoring Architecture

### Core Components

The monitoring system consists of:
- **CloudWatch Metrics**: System and application performance metrics
- **CloudWatch Alarms**: Automated alerting based on metric thresholds
- **CloudWatch Dashboards**: Visual monitoring interfaces
- **SNS Topics**: Alert notification distribution
- **CloudWatch Logs**: Centralized logging for all services

### Monitoring Construct

The monitoring infrastructure is defined in [`infrastructure/lib/monitoring-construct.ts`](frontend/src/../../infrastructure/lib/monitoring-construct.ts) and provides:

```typescript
export interface MonitoringConstructProps {
  tunnelInstance: ec2.Instance;
  environment: string;
  alertEmail?: string;
}

export class MonitoringConstruct extends Construct {
  public readonly dashboard: cloudwatch.Dashboard;
  public readonly alertTopic: sns.Topic;
}
```

## CloudWatch Metrics Configuration

### Lambda Function Metrics

#### Admin Function Monitoring (lines 350-380 in boat-listing-stack.ts)

**Error Rate Alarm**:
```typescript
const adminErrorAlarm = new cloudwatch.Alarm(this, 'AdminFunctionErrorAlarm', {
  alarmName: `admin-function-errors-${environment}`,
  alarmDescription: 'High error rate in admin Lambda function',
  metric: adminFunction.metricErrors({
    period: cdk.Duration.minutes(5),
    statistic: 'Sum',
  }),
  threshold: 5,
  evaluationPeriods: 2,
  treatMissingData: cloudwatch.TreatMissingData.NOT_BREACHING,
});
```

**Duration Alarm**:
```typescript
const adminDurationAlarm = new cloudwatch.Alarm(this, 'AdminFunctionDurationAlarm', {
  alarmName: `admin-function-duration-${environment}`,
  alarmDescription: 'High duration in admin Lambda function',
  metric: adminFunction.metricDuration({
    period: cdk.Duration.minutes(5),
    statistic: 'Average',
  }),
  threshold: 25000, // 25 seconds (close to 30s timeout)
  evaluationPeriods: 3,
  treatMissingData: cloudwatch.TreatMissingData.NOT_BREACHING,
});
```

**Throttle Alarm**:
```typescript
const adminThrottleAlarm = new cloudwatch.Alarm(this, 'AdminFunctionThrottleAlarm', {
  alarmName: `admin-function-throttles-${environment}`,
  alarmDescription: 'Admin Lambda function is being throttled',
  metric: adminFunction.metricThrottles({
    period: cdk.Duration.minutes(5),
    statistic: 'Sum',
  }),
  threshold: 1,
  evaluationPeriods: 1,
  treatMissingData: cloudwatch.TreatMissingData.NOT_BREACHING,
});
```

### DynamoDB Metrics

#### Table Throttle Monitoring

**Audit Logs Table**:
```typescript
const auditLogsThrottleAlarm = new cloudwatch.Alarm(this, 'AuditLogsThrottleAlarm', {
  alarmName: `audit-logs-throttles-${environment}`,
  alarmDescription: 'Audit logs table is being throttled',
  metric: new cloudwatch.Metric({
    namespace: 'AWS/DynamoDB',
    metricName: 'ThrottledRequests',
    dimensionsMap: {
      TableName: auditLogsTable.tableName,
    },
    statistic: 'Sum',
    period: cdk.Duration.minutes(5),
  }),
  threshold: 1,
  evaluationPeriods: 1,
  treatMissingData: cloudwatch.TreatMissingData.NOT_BREACHING,
});
```

**Admin Sessions Table**:
```typescript
const adminSessionsThrottleAlarm = new cloudwatch.Alarm(this, 'AdminSessionsThrottleAlarm', {
  alarmName: `admin-sessions-throttles-${environment}`,
  alarmDescription: 'Admin sessions table is being throttled',
  metric: new cloudwatch.Metric({
    namespace: 'AWS/DynamoDB',
    metricName: 'ThrottledRequests',
    dimensionsMap: {
      TableName: adminSessionsTable.tableName,
    },
    statistic: 'Sum',
    period: cdk.Duration.minutes(5),
  }),
  threshold: 1,
  evaluationPeriods: 1,
});
```

### EC2 Instance Metrics (Cloudflare Tunnel)

#### Instance Health Monitoring (lines 40-70 in monitoring-construct.ts)

**Instance Status Check**:
```typescript
const instanceStatusAlarm = new cloudwatch.Alarm(this, 'InstanceStatusAlarm', {
  alarmName: `cloudflare-tunnel-instance-status-${environment}`,
  alarmDescription: 'EC2 instance status check failed',
  metric: new cloudwatch.Metric({
    namespace: 'AWS/EC2',
    metricName: 'StatusCheckFailed',
    dimensionsMap: {
      InstanceId: tunnelInstance.instanceId,
    },
    statistic: 'Maximum',
    period: cdk.Duration.minutes(1),
  }),
  threshold: 1,
  evaluationPeriods: 2,
  treatMissingData: cloudwatch.TreatMissingData.BREACHING,
});
```

**System Status Check**:
```typescript
const systemStatusAlarm = new cloudwatch.Alarm(this, 'SystemStatusAlarm', {
  alarmName: `cloudflare-tunnel-system-status-${environment}`,
  alarmDescription: 'EC2 system status check failed',
  metric: new cloudwatch.Metric({
    namespace: 'AWS/EC2',
    metricName: 'StatusCheckFailed_System',
    dimensionsMap: {
      InstanceId: tunnelInstance.instanceId,
    },
    statistic: 'Maximum',
    period: cdk.Duration.minutes(1),
  }),
  threshold: 1,
  evaluationPeriods: 2,
});
```

**CPU Utilization**:
```typescript
const cpuAlarm = new cloudwatch.Alarm(this, 'HighCpuAlarm', {
  alarmName: `cloudflare-tunnel-high-cpu-${environment}`,
  alarmDescription: 'High CPU utilization on tunnel instance',
  metric: new cloudwatch.Metric({
    namespace: 'AWS/EC2',
    metricName: 'CPUUtilization',
    dimensionsMap: {
      InstanceId: tunnelInstance.instanceId,
    },
    statistic: 'Average',
    period: cdk.Duration.minutes(5),
  }),
  threshold: 80,
  evaluationPeriods: 3,
  treatMissingData: cloudwatch.TreatMissingData.NOT_BREACHING,
});
```

**Network Connectivity**:
```typescript
const networkAlarm = new cloudwatch.Alarm(this, 'NetworkConnectivityAlarm', {
  alarmName: `cloudflare-tunnel-network-${environment}`,
  alarmDescription: 'Low network activity - tunnel may be down',
  metric: new cloudwatch.Metric({
    namespace: 'AWS/EC2',
    metricName: 'NetworkPacketsOut',
    dimensionsMap: {
      InstanceId: tunnelInstance.instanceId,
    },
    statistic: 'Sum',
    period: cdk.Duration.minutes(5),
  }),
  threshold: 10,
  comparisonOperator: cloudwatch.ComparisonOperator.LESS_THAN_THRESHOLD,
  evaluationPeriods: 3,
  treatMissingData: cloudwatch.TreatMissingData.BREACHING,
});
```

## SNS Alert Configuration

### Alert Topics

#### Admin Service Alerts (lines 340-348 in boat-listing-stack.ts)
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

#### Monitoring Alerts (lines 25-35 in monitoring-construct.ts)
```typescript
this.alertTopic = new sns.Topic(this, 'MonitoringAlerts', {
  topicName: `cloudflare-tunnel-alerts-${environment}`,
  displayName: `Cloudflare Tunnel Monitoring Alerts - ${environment}`,
});

if (alertEmail) {
  this.alertTopic.addSubscription(
    new subscriptions.EmailSubscription(alertEmail)
  );
}
```

### Alert Actions

All alarms are configured to send notifications to their respective SNS topics:

```typescript
adminErrorAlarm.addAlarmAction(
  new cloudwatchActions.SnsAction(adminAlertTopic)
);

instanceStatusAlarm.addAlarmAction(
  new cloudwatchActions.SnsAction(this.alertTopic)
);
```

## CloudWatch Dashboards

### Admin Service Dashboard (lines 430-520 in boat-listing-stack.ts)

```typescript
const adminDashboard = new cloudwatch.Dashboard(this, 'AdminServiceDashboard', {
  dashboardName: `admin-service-${environment}`,
});

adminDashboard.addWidgets(
  // Lambda function metrics
  new cloudwatch.GraphWidget({
    title: 'Admin Function Performance',
    left: [
      adminFunction.metricInvocations({
        period: cdk.Duration.minutes(5),
        statistic: 'Sum',
        label: 'Invocations',
      }),
      adminFunction.metricErrors({
        period: cdk.Duration.minutes(5),
        statistic: 'Sum',
        label: 'Errors',
      }),
    ],
    right: [
      adminFunction.metricDuration({
        period: cdk.Duration.minutes(5),
        statistic: 'Average',
        label: 'Duration (ms)',
      }),
    ],
    width: 12,
    height: 6,
  })
);
```

### Infrastructure Monitoring Dashboard (lines 100-180 in monitoring-construct.ts)

```typescript
this.dashboard = new cloudwatch.Dashboard(this, 'MonitoringDashboard', {
  dashboardName: `cloudflare-tunnel-${environment}`,
});

this.dashboard.addWidgets(
  // Instance health overview
  new cloudwatch.GraphWidget({
    title: 'EC2 Instance Health',
    left: [
      new cloudwatch.Metric({
        namespace: 'AWS/EC2',
        metricName: 'StatusCheckFailed',
        dimensionsMap: {
          InstanceId: tunnelInstance.instanceId,
        },
        statistic: 'Maximum',
        period: cdk.Duration.minutes(1),
        label: 'Instance Status Check',
      }),
    ],
    width: 12,
    height: 6,
  }),

  // Resource utilization
  new cloudwatch.GraphWidget({
    title: 'Resource Utilization',
    left: [
      new cloudwatch.Metric({
        namespace: 'AWS/EC2',
        metricName: 'CPUUtilization',
        dimensionsMap: {
          InstanceId: tunnelInstance.instanceId,
        },
        statistic: 'Average',
        period: cdk.Duration.minutes(5),
        label: 'CPU Utilization (%)',
      }),
    ],
    width: 12,
    height: 6,
  })
);
```

## CloudWatch Logs Configuration

### Log Groups

#### Tunnel Service Logs (lines 40-45 in monitoring-construct.ts)
```typescript
const tunnelLogGroup = new logs.LogGroup(this, 'TunnelServiceLogs', {
  logGroupName: `/aws/ec2/cloudflare-tunnel/${environment}`,
  retention: logs.RetentionDays.ONE_WEEK,
  removalPolicy: cdk.RemovalPolicy.DESTROY,
});
```

#### Lambda Function Logs
Lambda functions automatically create log groups with the pattern:
- `/aws/lambda/BoatListingStack-${environment}-AdminFunction-*`
- `/aws/lambda/BoatListingStack-${environment}-ListingFunction-*`
- `/aws/lambda/BoatListingStack-${environment}-AuthFunction-*`

### Log Retention Policies

- **Development**: 1 week retention for cost optimization
- **Staging**: 2 weeks retention for debugging
- **Production**: 30 days retention for compliance and troubleshooting

## Monitoring Procedures

### Daily Monitoring Checklist

1. **Check Dashboard Status**:
   - Review admin service dashboard for anomalies
   - Monitor infrastructure dashboard for resource utilization
   - Verify all systems are operational

2. **Review Alerts**:
   - Check for any overnight alerts
   - Investigate and resolve any active alarms
   - Update alert thresholds if needed

3. **Log Analysis**:
   - Review error logs for patterns
   - Check for security-related events
   - Monitor performance trends

### Weekly Monitoring Tasks

1. **Performance Review**:
   - Analyze Lambda function performance trends
   - Review DynamoDB capacity utilization
   - Check S3 storage growth patterns

2. **Cost Analysis**:
   - Review CloudWatch costs and usage
   - Optimize log retention policies
   - Adjust monitoring frequency if needed

3. **Alert Tuning**:
   - Review false positive alerts
   - Adjust thresholds based on usage patterns
   - Add new alerts for emerging issues

### Monthly Monitoring Tasks

1. **Comprehensive Review**:
   - Full system performance analysis
   - Security monitoring review
   - Capacity planning assessment

2. **Documentation Updates**:
   - Update monitoring procedures
   - Review and update alert escalation procedures
   - Update dashboard configurations

## Alert Escalation Procedures

### Severity Levels

#### Critical (P1) - Immediate Response Required
- **Triggers**: System down, data loss, security breach
- **Response Time**: 15 minutes
- **Escalation**: Immediate notification to on-call engineer
- **Examples**:
  - All Lambda functions failing
  - DynamoDB tables inaccessible
  - Security alerts from login attempts

#### High (P2) - Urgent Response Required
- **Triggers**: Performance degradation, partial service outage
- **Response Time**: 1 hour
- **Escalation**: Notification to development team
- **Examples**:
  - High error rates (>5% of requests)
  - Lambda function timeouts
  - High CPU utilization (>80%)

#### Medium (P3) - Standard Response
- **Triggers**: Minor performance issues, warnings
- **Response Time**: 4 hours
- **Escalation**: Standard ticket creation
- **Examples**:
  - Moderate error rates (1-5% of requests)
  - Approaching resource limits
  - Non-critical service degradation

#### Low (P4) - Informational
- **Triggers**: Informational alerts, trend notifications
- **Response Time**: Next business day
- **Escalation**: Log for review
- **Examples**:
  - Usage pattern changes
  - Capacity planning alerts
  - Performance optimization opportunities

### Escalation Workflow

1. **Initial Alert**: SNS notification sent to configured email addresses
2. **Acknowledgment**: Responder acknowledges alert within SLA timeframe
3. **Investigation**: Initial investigation and impact assessment
4. **Resolution**: Implement fix or escalate to next level
5. **Post-Incident**: Document resolution and update procedures

## Monitoring Commands and Scripts

### CloudWatch CLI Commands

#### View Recent Alarms
```bash
# List all alarms in ALARM state
aws cloudwatch describe-alarms \
  --state-value ALARM \
  --region us-east-1

# Get specific alarm history
aws cloudwatch describe-alarm-history \
  --alarm-name "admin-function-errors-prod" \
  --region us-east-1
```

#### Query Metrics
```bash
# Get Lambda function metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Invocations \
  --dimensions Name=FunctionName,Value=BoatListingStack-prod-AdminFunction \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-01T23:59:59Z \
  --period 3600 \
  --statistics Sum \
  --region us-east-1

# Get DynamoDB metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/DynamoDB \
  --metric-name ConsumedReadCapacityUnits \
  --dimensions Name=TableName,Value=boat-listings \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-01T23:59:59Z \
  --period 3600 \
  --statistics Sum \
  --region us-east-1
```

#### Log Analysis
```bash
# Filter Lambda function logs
aws logs filter-log-events \
  --log-group-name "/aws/lambda/BoatListingStack-prod-AdminFunction" \
  --start-time 1640995200000 \
  --filter-pattern "ERROR" \
  --region us-east-1

# Tail logs in real-time
aws logs tail "/aws/lambda/BoatListingStack-prod-AdminFunction" \
  --follow \
  --region us-east-1
```

### Custom Monitoring Scripts

#### Health Check Script
```bash
#!/bin/bash
# infrastructure/scripts/health-check.sh

ENVIRONMENT=${1:-dev}
API_URL=$(aws cloudformation describe-stacks \
  --stack-name "BoatListingStack-${ENVIRONMENT}" \
  --query 'Stacks[0].Outputs[?OutputKey==`ApiUrl`].OutputValue' \
  --output text \
  --region us-east-1)

# Test API health
if curl -f -s "${API_URL}/health" > /dev/null; then
  echo "✅ API is healthy"
else
  echo "❌ API health check failed"
  exit 1
fi

# Test database connectivity
if aws dynamodb describe-table \
  --table-name "boat-listings" \
  --region us-east-1 > /dev/null 2>&1; then
  echo "✅ Database is accessible"
else
  echo "❌ Database connectivity failed"
  exit 1
fi
```

#### Performance Monitoring Script
```bash
#!/bin/bash
# infrastructure/scripts/performance-monitor.sh

ENVIRONMENT=${1:-dev}
FUNCTION_NAME="BoatListingStack-${ENVIRONMENT}-AdminFunction"

# Get function metrics for the last hour
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Duration \
  --dimensions Name=FunctionName,Value="${FUNCTION_NAME}" \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%SZ) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
  --period 300 \
  --statistics Average,Maximum \
  --region us-east-1
```

## Dashboard URLs and Access

### CloudWatch Dashboard URLs

The infrastructure outputs provide direct links to monitoring dashboards:

```typescript
// Admin Service Dashboard URL
new cdk.CfnOutput(this, 'AdminDashboardUrl', {
  value: `https://${cdk.Stack.of(this).region}.console.aws.amazon.com/cloudwatch/home?region=${cdk.Stack.of(this).region}#dashboards:name=${adminDashboard.dashboardName}`,
  description: 'Admin Service CloudWatch Dashboard URL',
});

// Infrastructure Monitoring Dashboard URL
new cdk.CfnOutput(this, 'DashboardUrl', {
  value: `https://${cdk.Stack.of(this).region}.console.aws.amazon.com/cloudwatch/home?region=${cdk.Stack.of(this).region}#dashboards:name=${this.dashboard.dashboardName}`,
  description: 'CloudWatch Dashboard URL',
});
```

### Access Management

Dashboard access is controlled through AWS IAM permissions:
- **ReadOnly Access**: CloudWatch read permissions for monitoring team
- **Full Access**: CloudWatch full permissions for operations team
- **Admin Access**: Full AWS permissions for system administrators

## Performance Baselines and Thresholds

### Lambda Function Baselines

#### Admin Function
- **Normal Duration**: 100-500ms
- **Warning Threshold**: 1000ms average over 5 minutes
- **Critical Threshold**: 25000ms (near timeout)
- **Error Rate**: <1% normal, >5% critical
- **Invocation Rate**: 10-100 per minute normal

#### Listing Function
- **Normal Duration**: 50-200ms
- **Warning Threshold**: 500ms average over 5 minutes
- **Critical Threshold**: 15000ms
- **Error Rate**: <0.5% normal, >2% critical
- **Invocation Rate**: 50-500 per minute normal

### DynamoDB Baselines

#### Tables Performance
- **Read Latency**: <10ms average
- **Write Latency**: <15ms average
- **Throttled Requests**: 0 (any throttling triggers alert)
- **Consumed Capacity**: Monitor for auto-scaling triggers

### Infrastructure Baselines

#### EC2 Instance (Cloudflare Tunnel)
- **CPU Utilization**: 5-20% normal, >80% warning
- **Network Activity**: Consistent packet flow expected
- **Status Checks**: Must always pass

## Troubleshooting Common Issues

### High Lambda Error Rates

1. **Check Recent Deployments**: Correlate with deployment times
2. **Review Error Logs**: Analyze specific error patterns
3. **Check Dependencies**: Verify DynamoDB and external service health
4. **Resource Limits**: Check memory and timeout configurations

### DynamoDB Throttling

1. **Check Capacity**: Review consumed vs. provisioned capacity
2. **Hot Partitions**: Analyze access patterns for hot keys
3. **GSI Usage**: Verify Global Secondary Index utilization
4. **Burst Capacity**: Check if burst capacity is exhausted

### EC2 Instance Issues

1. **Status Checks**: Verify both instance and system status
2. **Resource Utilization**: Check CPU, memory, and network usage
3. **Cloudflare Tunnel**: Verify tunnel connectivity and configuration
4. **Security Groups**: Check network access rules

### Alert Fatigue Management

1. **Threshold Tuning**: Adjust thresholds based on actual usage patterns
2. **Alert Grouping**: Combine related alerts to reduce noise
3. **Maintenance Windows**: Suppress alerts during planned maintenance
4. **False Positive Analysis**: Regular review and adjustment of alert conditions

## Integration with External Monitoring

### Cloudflare Analytics

The platform integrates with Cloudflare for additional monitoring:
- **Traffic Analytics**: Request volume and geographic distribution
- **Security Events**: DDoS attacks and security threats
- **Performance Metrics**: Cache hit ratios and response times
- **Uptime Monitoring**: Global availability monitoring

### Third-Party Integration Options

#### Datadog Integration
```typescript
// Optional Datadog integration
const datadogIntegration = new datadog.DatadogIntegration(this, 'DatadogIntegration', {
  apiKey: datadogApiKey,
  environment: environment,
  service: 'marine-market',
});
```

#### New Relic Integration
```typescript
// Optional New Relic integration
const newRelicIntegration = new newrelic.NewRelicIntegration(this, 'NewRelicIntegration', {
  licenseKey: newRelicLicenseKey,
  applicationName: `marine-market-${environment}`,
});
```

## Monitoring Best Practices

### Metric Collection

1. **Business Metrics**: Track user registrations, listings created, searches performed
2. **Technical Metrics**: Response times, error rates, resource utilization
3. **Security Metrics**: Failed login attempts, suspicious activities
4. **Cost Metrics**: Resource usage and billing trends

### Alert Design

1. **Actionable Alerts**: Every alert should have a clear response procedure
2. **Appropriate Thresholds**: Based on historical data and business requirements
3. **Alert Fatigue Prevention**: Avoid too many low-priority alerts
4. **Context Information**: Include relevant details for quick diagnosis

### Dashboard Design

1. **Role-Based Views**: Different dashboards for different audiences
2. **Hierarchical Information**: High-level overview with drill-down capability
3. **Real-Time Data**: Balance between real-time updates and performance
4. **Mobile Friendly**: Ensure dashboards work on mobile devices

## Maintenance and Updates

### Regular Maintenance Tasks

#### Weekly
- Review and acknowledge all alerts
- Check dashboard functionality
- Update alert thresholds if needed
- Review log retention policies

#### Monthly
- Comprehensive performance review
- Update monitoring documentation
- Review and test alert escalation procedures
- Analyze monitoring costs and optimize

#### Quarterly
- Full monitoring system audit
- Update monitoring strategy based on system changes
- Review and update SLAs and SLOs
- Conduct monitoring system disaster recovery testing

### Monitoring System Updates

1. **CDK Updates**: Keep monitoring construct updated with latest CDK version
2. **Metric Updates**: Add new metrics as system evolves
3. **Dashboard Updates**: Enhance dashboards based on user feedback
4. **Alert Updates**: Refine alerts based on operational experience

## Related Documentation

- [Infrastructure Setup Guide](infrastructure-setup.md)
- [Operational Runbooks](../operations/)
- [Security Monitoring](../security/)
- [Performance Testing Guide](testing/performance-testing.md)
- [Cost Optimization Guide](cost-optimization.md)