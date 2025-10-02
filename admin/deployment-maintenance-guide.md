# Admin Dashboard Deployment and Maintenance Guide

## Overview

This guide provides comprehensive instructions for deploying, maintaining, and operating the HarborList Admin Dashboard in production environments. It covers deployment procedures, monitoring, maintenance tasks, troubleshooting, and operational best practices.

## Table of Contents

1. [Deployment Prerequisites](#deployment-prerequisites)
2. [Initial Deployment](#initial-deployment)
3. [Environment Configuration](#environment-configuration)
4. [Database Setup](#database-setup)
5. [Security Configuration](#security-configuration)
6. [Monitoring Setup](#monitoring-setup)
7. [Backup and Recovery](#backup-and-recovery)
8. [Maintenance Procedures](#maintenance-procedures)
9. [Troubleshooting](#troubleshooting)
10. [Performance Optimization](#performance-optimization)
11. [Disaster Recovery](#disaster-recovery)

## Deployment Prerequisites

### 1.1 System Requirements

**Minimum Requirements:**
- **CPU**: 4 vCPUs
- **Memory**: 8 GB RAM
- **Storage**: 100 GB SSD
- **Network**: 1 Gbps connection
- **OS**: Ubuntu 20.04 LTS or Amazon Linux 2

**Recommended Requirements:**
- **CPU**: 8 vCPUs
- **Memory**: 16 GB RAM
- **Storage**: 200 GB SSD with backup storage
- **Network**: 10 Gbps connection
- **Load Balancer**: Application Load Balancer
- **CDN**: CloudFront distribution

### 1.2 Software Dependencies

**Required Software:**
```bash
# Node.js and npm
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Docker (for containerized deployment)
sudo apt-get update
sudo apt-get install -y docker.io docker-compose

# Nginx (for reverse proxy)
sudo apt-get install -y nginx

# SSL certificates
sudo apt-get install -y certbot python3-certbot-nginx
```

**Development Tools:**
```bash
# Git
sudo apt-get install -y git

# Build tools
sudo apt-get install -y build-essential

# Monitoring tools
sudo apt-get install -y htop iotop nethogs
```

### 1.3 AWS Services Setup

**Required AWS Services:**
- [ ] **EC2**: Application hosting
- [ ] **RDS**: Database hosting
- [ ] **ElastiCache**: Redis caching
- [ ] **S3**: File storage and backups
- [ ] **CloudFront**: CDN
- [ ] **Route 53**: DNS management
- [ ] **Certificate Manager**: SSL certificates
- [ ] **CloudWatch**: Monitoring and logging
- [ ] **IAM**: Access management

**IAM Roles and Policies:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:UpdateItem",
        "dynamodb:DeleteItem",
        "dynamodb:Query",
        "dynamodb:Scan"
      ],
      "Resource": [
        "arn:aws:dynamodb:*:*:table/admin-*",
        "arn:aws:dynamodb:*:*:table/audit-*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": [
        "arn:aws:s3:::harbotlist-admin/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "cloudwatch:PutMetricData",
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "*"
    }
  ]
}
```

## Initial Deployment

### 2.1 Infrastructure Deployment

**Using AWS CDK:**
```bash
# Clone the repository
git clone https://github.com/harbotlist/admin-dashboard.git
cd admin-dashboard

# Install dependencies
npm install

# Deploy infrastructure
cd infrastructure
npm install
npx cdk bootstrap
npx cdk deploy --all
```

**Manual Infrastructure Setup:**
```bash
# Create VPC and subnets
aws ec2 create-vpc --cidr-block 10.0.0.0/16
aws ec2 create-subnet --vpc-id vpc-xxx --cidr-block 10.0.1.0/24
aws ec2 create-subnet --vpc-id vpc-xxx --cidr-block 10.0.2.0/24

# Create security groups
aws ec2 create-security-group --group-name admin-sg --description "Admin Dashboard Security Group"

# Create RDS instance
aws rds create-db-instance \
  --db-instance-identifier admin-db \
  --db-instance-class db.t3.medium \
  --engine postgres \
  --master-username admin \
  --master-user-password SecurePassword123! \
  --allocated-storage 100
```

### 2.2 Application Deployment

**Backend Deployment:**
```bash
# Build backend
cd backend
npm install
npm run build

# Deploy to Lambda
npm run deploy:prod

# Or deploy to EC2
pm2 start dist/index.js --name admin-api
```

**Frontend Deployment:**
```bash
# Build frontend
cd frontend
npm install
npm run build

# Deploy to S3
aws s3 sync dist/ s3://harbotlist-admin-frontend/

# Invalidate CloudFront cache
aws cloudfront create-invalidation \
  --distribution-id E1234567890123 \
  --paths "/*"
```

### 2.3 Database Migration

**Run Database Migrations:**
```bash
# Create tables
npm run db:migrate

# Seed initial data
npm run db:seed

# Create admin user
npm run create-admin-user -- \
  --email admin@harbotlist.com \
  --name "System Administrator" \
  --password SecurePassword123!
```

## Environment Configuration

### 3.1 Environment Variables

**Backend Environment Variables:**
```bash
# .env.production
NODE_ENV=production
PORT=3000

# Database
DATABASE_URL=postgresql://admin:password@admin-db.region.rds.amazonaws.com:5432/admin
REDIS_URL=redis://admin-cache.region.cache.amazonaws.com:6379

# JWT Configuration
JWT_SECRET=your-super-secure-jwt-secret-key
JWT_EXPIRES_IN=1h
JWT_REFRESH_EXPIRES_IN=7d

# AWS Configuration
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=...

# DynamoDB Tables
ADMIN_TABLE=admin-users-prod
AUDIT_TABLE=audit-logs-prod
SESSIONS_TABLE=admin-sessions-prod

# Email Configuration
SMTP_HOST=email-smtp.us-east-1.amazonaws.com
SMTP_PORT=587
SMTP_USER=AKIA...
SMTP_PASS=...
FROM_EMAIL=noreply@harbotlist.com

# Monitoring
SENTRY_DSN=https://...@sentry.io/...
NEW_RELIC_LICENSE_KEY=...

# Security
RATE_LIMIT_WINDOW=15
RATE_LIMIT_MAX=100
SESSION_TIMEOUT=28800
CORS_ORIGIN=https://admin.harbotlist.com
```

**Frontend Environment Variables:**
```bash
# .env.production
VITE_API_URL=https://api.harbotlist.com/admin
VITE_APP_NAME=HarborList Admin
VITE_APP_VERSION=1.0.0
VITE_SENTRY_DSN=https://...@sentry.io/...
VITE_ANALYTICS_ID=GA_MEASUREMENT_ID
```

### 3.2 Configuration Management

**Using AWS Systems Manager Parameter Store:**
```bash
# Store sensitive configuration
aws ssm put-parameter \
  --name "/harbotlist/admin/jwt-secret" \
  --value "your-jwt-secret" \
  --type "SecureString"

aws ssm put-parameter \
  --name "/harbotlist/admin/database-url" \
  --value "postgresql://..." \
  --type "SecureString"
```

**Configuration Loading Script:**
```javascript
// config/loader.js
const AWS = require('aws-sdk');
const ssm = new AWS.SSM();

async function loadConfig() {
  const parameters = await ssm.getParameters({
    Names: [
      '/harbotlist/admin/jwt-secret',
      '/harbotlist/admin/database-url',
      '/harbotlist/admin/redis-url'
    ],
    WithDecryption: true
  }).promise();

  const config = {};
  parameters.Parameters.forEach(param => {
    const key = param.Name.split('/').pop().replace('-', '_').toUpperCase();
    config[key] = param.Value;
  });

  return config;
}
```

## Database Setup

### 4.1 Database Schema

**Create Database Schema:**
```sql
-- Create admin users table
CREATE TABLE admin_users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  name VARCHAR(255) NOT NULL,
  role VARCHAR(50) NOT NULL DEFAULT 'admin',
  permissions TEXT[] DEFAULT '{}',
  is_active BOOLEAN DEFAULT true,
  mfa_enabled BOOLEAN DEFAULT false,
  mfa_secret VARCHAR(255),
  last_login TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create audit logs table
CREATE TABLE audit_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  admin_id UUID REFERENCES admin_users(id),
  action VARCHAR(255) NOT NULL,
  resource VARCHAR(255) NOT NULL,
  resource_id VARCHAR(255),
  details JSONB,
  ip_address INET,
  user_agent TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create sessions table
CREATE TABLE admin_sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  admin_id UUID REFERENCES admin_users(id),
  session_token VARCHAR(255) UNIQUE NOT NULL,
  device_id VARCHAR(255),
  ip_address INET,
  user_agent TEXT,
  expires_at TIMESTAMP NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  last_activity TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create indexes
CREATE INDEX idx_audit_logs_admin_id ON audit_logs(admin_id);
CREATE INDEX idx_audit_logs_created_at ON audit_logs(created_at);
CREATE INDEX idx_audit_logs_action ON audit_logs(action);
CREATE INDEX idx_sessions_admin_id ON admin_sessions(admin_id);
CREATE INDEX idx_sessions_expires_at ON admin_sessions(expires_at);
```

### 4.2 Database Maintenance

**Automated Backup Script:**
```bash
#!/bin/bash
# backup-database.sh

DB_HOST="admin-db.region.rds.amazonaws.com"
DB_NAME="admin"
DB_USER="admin"
BACKUP_DIR="/var/backups/admin-db"
DATE=$(date +%Y%m%d_%H%M%S)

# Create backup directory
mkdir -p $BACKUP_DIR

# Create database backup
pg_dump -h $DB_HOST -U $DB_USER -d $DB_NAME \
  --no-password --clean --create \
  > $BACKUP_DIR/admin_backup_$DATE.sql

# Compress backup
gzip $BACKUP_DIR/admin_backup_$DATE.sql

# Upload to S3
aws s3 cp $BACKUP_DIR/admin_backup_$DATE.sql.gz \
  s3://harbotlist-backups/database/

# Clean up old local backups (keep 7 days)
find $BACKUP_DIR -name "*.sql.gz" -mtime +7 -delete

# Clean up old S3 backups (keep 30 days)
aws s3 ls s3://harbotlist-backups/database/ \
  --query 'Contents[?LastModified<=`2024-08-29`].Key' \
  --output text | xargs -I {} aws s3 rm s3://harbotlist-backups/database/{}
```

**Database Monitoring Script:**
```sql
-- Monitor database performance
SELECT 
  schemaname,
  tablename,
  n_tup_ins as inserts,
  n_tup_upd as updates,
  n_tup_del as deletes,
  n_live_tup as live_tuples,
  n_dead_tup as dead_tuples
FROM pg_stat_user_tables
ORDER BY n_live_tup DESC;

-- Monitor slow queries
SELECT 
  query,
  calls,
  total_time,
  mean_time,
  rows
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;
```

## Security Configuration

### 5.1 SSL/TLS Configuration

**Nginx SSL Configuration:**
```nginx
# /etc/nginx/sites-available/admin.harbotlist.com
server {
    listen 80;
    server_name admin.harbotlist.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name admin.harbotlist.com;

    # SSL Configuration
    ssl_certificate /etc/letsencrypt/live/admin.harbotlist.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/admin.harbotlist.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    # Security Headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self' data:; connect-src 'self' https://api.harbotlist.com;" always;

    # Rate Limiting
    limit_req_zone $binary_remote_addr zone=admin:10m rate=10r/m;
    limit_req zone=admin burst=20 nodelay;

    location / {
        root /var/www/admin;
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 5.2 Firewall Configuration

**UFW Firewall Rules:**
```bash
# Enable UFW
sudo ufw enable

# Allow SSH (change port if needed)
sudo ufw allow 22/tcp

# Allow HTTP and HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Allow specific IPs for admin access
sudo ufw allow from 203.0.113.0/24 to any port 22
sudo ufw allow from 203.0.113.0/24 to any port 443

# Deny all other traffic
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

**AWS Security Group Rules:**
```bash
# Create security group
aws ec2 create-security-group \
  --group-name admin-dashboard-sg \
  --description "Admin Dashboard Security Group"

# Allow HTTPS from anywhere
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxx \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# Allow SSH from office IP
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxx \
  --protocol tcp \
  --port 22 \
  --cidr 203.0.113.0/24
```

### 5.3 Access Control

**IP Whitelisting Configuration:**
```javascript
// middleware/ipWhitelist.js
const allowedIPs = [
  '203.0.113.0/24',  // Office network
  '198.51.100.0/24', // VPN network
  '192.0.2.0/24'     // Admin network
];

function ipWhitelist(req, res, next) {
  const clientIP = req.ip || req.connection.remoteAddress;
  
  const isAllowed = allowedIPs.some(range => {
    return ipRangeCheck(clientIP, range);
  });
  
  if (!isAllowed) {
    return res.status(403).json({ error: 'Access denied' });
  }
  
  next();
}
```

## Monitoring Setup

### 6.1 Application Monitoring

**CloudWatch Metrics:**
```javascript
// monitoring/metrics.js
const AWS = require('aws-sdk');
const cloudwatch = new AWS.CloudWatch();

class MetricsCollector {
  async recordMetric(metricName, value, unit = 'Count') {
    const params = {
      Namespace: 'HarborList/Admin',
      MetricData: [{
        MetricName: metricName,
        Value: value,
        Unit: unit,
        Timestamp: new Date()
      }]
    };
    
    await cloudwatch.putMetricData(params).promise();
  }
  
  async recordLoginAttempt(success) {
    await this.recordMetric('LoginAttempts', 1);
    await this.recordMetric(success ? 'SuccessfulLogins' : 'FailedLogins', 1);
  }
  
  async recordAPICall(endpoint, responseTime) {
    await this.recordMetric(`API.${endpoint}.Calls`, 1);
    await this.recordMetric(`API.${endpoint}.ResponseTime`, responseTime, 'Milliseconds');
  }
}
```

**Health Check Endpoint:**
```javascript
// routes/health.js
app.get('/health', async (req, res) => {
  const health = {
    status: 'healthy',
    timestamp: new Date().toISOString(),
    version: process.env.APP_VERSION,
    checks: {}
  };
  
  try {
    // Database check
    await db.query('SELECT 1');
    health.checks.database = 'healthy';
  } catch (error) {
    health.checks.database = 'unhealthy';
    health.status = 'unhealthy';
  }
  
  try {
    // Redis check
    await redis.ping();
    health.checks.redis = 'healthy';
  } catch (error) {
    health.checks.redis = 'unhealthy';
    health.status = 'unhealthy';
  }
  
  const statusCode = health.status === 'healthy' ? 200 : 503;
  res.status(statusCode).json(health);
});
```

### 6.2 Log Management

**Structured Logging Configuration:**
```javascript
// config/logger.js
const winston = require('winston');
const { ElasticsearchTransport } = require('winston-elasticsearch');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: { service: 'admin-dashboard' },
  transports: [
    new winston.transports.File({ filename: 'logs/error.log', level: 'error' }),
    new winston.transports.File({ filename: 'logs/combined.log' }),
    new ElasticsearchTransport({
      level: 'info',
      clientOpts: { node: 'https://elasticsearch.harbotlist.com' },
      index: 'admin-logs'
    })
  ]
});

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }));
}
```

### 6.3 Alerting Configuration

**CloudWatch Alarms:**
```bash
# High error rate alarm
aws cloudwatch put-metric-alarm \
  --alarm-name "Admin-High-Error-Rate" \
  --alarm-description "High error rate in admin dashboard" \
  --metric-name "ErrorRate" \
  --namespace "HarborList/Admin" \
  --statistic "Average" \
  --period 300 \
  --threshold 5.0 \
  --comparison-operator "GreaterThanThreshold" \
  --evaluation-periods 2 \
  --alarm-actions "arn:aws:sns:us-east-1:123456789012:admin-alerts"

# Failed login attempts alarm
aws cloudwatch put-metric-alarm \
  --alarm-name "Admin-Failed-Logins" \
  --alarm-description "High number of failed login attempts" \
  --metric-name "FailedLogins" \
  --namespace "HarborList/Admin" \
  --statistic "Sum" \
  --period 300 \
  --threshold 10 \
  --comparison-operator "GreaterThanThreshold" \
  --evaluation-periods 1 \
  --alarm-actions "arn:aws:sns:us-east-1:123456789012:security-alerts"
```

## Backup and Recovery

### 7.1 Backup Strategy

**Automated Backup Script:**
```bash
#!/bin/bash
# backup-admin-system.sh

BACKUP_DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/var/backups/admin-system"
S3_BUCKET="harbotlist-backups"

# Create backup directory
mkdir -p $BACKUP_DIR/$BACKUP_DATE

# Database backup
pg_dump -h $DB_HOST -U $DB_USER -d $DB_NAME \
  > $BACKUP_DIR/$BACKUP_DATE/database.sql

# Application files backup
tar -czf $BACKUP_DIR/$BACKUP_DATE/application.tar.gz \
  /opt/admin-dashboard \
  /etc/nginx/sites-available/admin.harbotlist.com \
  /etc/ssl/certs/admin.harbotlist.com

# Configuration backup
cp -r /opt/admin-dashboard/.env* $BACKUP_DIR/$BACKUP_DATE/
cp -r /etc/systemd/system/admin-* $BACKUP_DIR/$BACKUP_DATE/

# Upload to S3
aws s3 sync $BACKUP_DIR/$BACKUP_DATE \
  s3://$S3_BUCKET/admin-system/$BACKUP_DATE/

# Clean up old backups
find $BACKUP_DIR -type d -mtime +7 -exec rm -rf {} \;
```

### 7.2 Recovery Procedures

**Database Recovery:**
```bash
#!/bin/bash
# restore-database.sh

BACKUP_FILE=$1
DB_HOST="admin-db.region.rds.amazonaws.com"
DB_NAME="admin"
DB_USER="admin"

if [ -z "$BACKUP_FILE" ]; then
  echo "Usage: $0 <backup_file>"
  exit 1
fi

# Download backup from S3 if needed
if [[ $BACKUP_FILE == s3://* ]]; then
  aws s3 cp $BACKUP_FILE ./database_restore.sql
  BACKUP_FILE="./database_restore.sql"
fi

# Stop application
sudo systemctl stop admin-dashboard

# Restore database
psql -h $DB_HOST -U $DB_USER -d $DB_NAME < $BACKUP_FILE

# Start application
sudo systemctl start admin-dashboard

# Verify restoration
curl -f https://admin.harbotlist.com/health || echo "Health check failed"
```

**Application Recovery:**
```bash
#!/bin/bash
# restore-application.sh

BACKUP_DATE=$1
S3_BUCKET="harbotlist-backups"

if [ -z "$BACKUP_DATE" ]; then
  echo "Usage: $0 <backup_date>"
  exit 1
fi

# Stop services
sudo systemctl stop admin-dashboard
sudo systemctl stop nginx

# Download backup
aws s3 sync s3://$S3_BUCKET/admin-system/$BACKUP_DATE/ \
  /tmp/restore/

# Restore application files
tar -xzf /tmp/restore/application.tar.gz -C /

# Restore configuration
cp /tmp/restore/.env* /opt/admin-dashboard/

# Restore nginx configuration
cp /tmp/restore/admin.harbotlist.com \
  /etc/nginx/sites-available/

# Start services
sudo systemctl start nginx
sudo systemctl start admin-dashboard

# Verify restoration
sleep 10
curl -f https://admin.harbotlist.com/health
```

## Maintenance Procedures

### 8.1 Regular Maintenance Tasks

**Daily Tasks:**
```bash
#!/bin/bash
# daily-maintenance.sh

# Check system health
curl -f https://admin.harbotlist.com/health

# Check disk space
df -h | awk '$5 > 80 {print "Warning: " $0}'

# Check memory usage
free -m | awk 'NR==2{printf "Memory Usage: %s/%sMB (%.2f%%)\n", $3,$2,$3*100/$2 }'

# Check log file sizes
find /var/log -name "*.log" -size +100M -exec ls -lh {} \;

# Rotate logs if needed
sudo logrotate -f /etc/logrotate.conf

# Update security patches
sudo apt update && sudo apt list --upgradable | grep -i security
```

**Weekly Tasks:**
```bash
#!/bin/bash
# weekly-maintenance.sh

# Full system backup
./backup-admin-system.sh

# Database maintenance
psql -h $DB_HOST -U $DB_USER -d $DB_NAME -c "VACUUM ANALYZE;"

# Clean up old sessions
psql -h $DB_HOST -U $DB_USER -d $DB_NAME -c "
  DELETE FROM admin_sessions 
  WHERE expires_at < NOW() - INTERVAL '7 days';"

# Clean up old audit logs (keep 1 year)
psql -h $DB_HOST -U $DB_USER -d $DB_NAME -c "
  DELETE FROM audit_logs 
  WHERE created_at < NOW() - INTERVAL '1 year';"

# Check SSL certificate expiration
openssl x509 -in /etc/letsencrypt/live/admin.harbotlist.com/cert.pem \
  -noout -dates

# Update dependencies
cd /opt/admin-dashboard
npm audit
npm update
```

**Monthly Tasks:**
```bash
#!/bin/bash
# monthly-maintenance.sh

# Security audit
./security-audit.sh

# Performance review
./performance-review.sh

# Capacity planning review
./capacity-review.sh

# Update documentation
git pull origin main
```

### 8.2 Update Procedures

**Application Update Process:**
```bash
#!/bin/bash
# update-application.sh

VERSION=$1

if [ -z "$VERSION" ]; then
  echo "Usage: $0 <version>"
  exit 1
fi

# Create backup before update
./backup-admin-system.sh

# Download new version
git fetch origin
git checkout $VERSION

# Install dependencies
npm install

# Run database migrations
npm run db:migrate

# Build application
npm run build

# Run tests
npm test

# Deploy new version
pm2 reload admin-api

# Verify deployment
sleep 10
curl -f https://admin.harbotlist.com/health

# Update frontend
cd frontend
npm run build
aws s3 sync dist/ s3://harbotlist-admin-frontend/
aws cloudfront create-invalidation \
  --distribution-id E1234567890123 \
  --paths "/*"
```

## Troubleshooting

### 9.1 Common Issues

**Issue: Application Won't Start**
```bash
# Check logs
sudo journalctl -u admin-dashboard -f

# Check configuration
node -e "console.log(require('./config'))"

# Check dependencies
npm ls

# Check ports
sudo netstat -tlnp | grep :3000

# Check permissions
ls -la /opt/admin-dashboard
```

**Issue: Database Connection Errors**
```bash
# Test database connection
psql -h $DB_HOST -U $DB_USER -d $DB_NAME -c "SELECT 1;"

# Check database status
aws rds describe-db-instances --db-instance-identifier admin-db

# Check security groups
aws ec2 describe-security-groups --group-ids sg-xxx

# Check network connectivity
telnet $DB_HOST 5432
```

**Issue: High Memory Usage**
```bash
# Check memory usage by process
ps aux --sort=-%mem | head -10

# Check for memory leaks
node --inspect /opt/admin-dashboard/dist/index.js

# Monitor memory over time
while true; do
  free -m | awk 'NR==2{printf "%s: %s/%sMB (%.2f%%)\n", strftime("%Y-%m-%d %H:%M:%S"), $3,$2,$3*100/$2 }'
  sleep 60
done
```

### 9.2 Performance Issues

**Slow Database Queries:**
```sql
-- Find slow queries
SELECT 
  query,
  calls,
  total_time,
  mean_time,
  rows
FROM pg_stat_statements
WHERE mean_time > 1000
ORDER BY mean_time DESC;

-- Check for missing indexes
SELECT 
  schemaname,
  tablename,
  attname,
  n_distinct,
  correlation
FROM pg_stats
WHERE schemaname = 'public'
  AND n_distinct > 100
  AND correlation < 0.1;
```

**High CPU Usage:**
```bash
# Find CPU-intensive processes
top -o %CPU

# Profile Node.js application
node --prof /opt/admin-dashboard/dist/index.js

# Analyze profile
node --prof-process isolate-*.log > profile.txt
```

### 9.3 Security Incidents

**Suspicious Login Activity:**
```sql
-- Check failed login attempts
SELECT 
  ip_address,
  COUNT(*) as attempts,
  MAX(created_at) as last_attempt
FROM audit_logs
WHERE action = 'login_failed'
  AND created_at > NOW() - INTERVAL '1 hour'
GROUP BY ip_address
HAVING COUNT(*) > 5
ORDER BY attempts DESC;
```

**Incident Response Checklist:**
- [ ] Identify the scope of the incident
- [ ] Isolate affected systems
- [ ] Preserve evidence
- [ ] Notify stakeholders
- [ ] Implement containment measures
- [ ] Eradicate the threat
- [ ] Recover systems
- [ ] Document lessons learned

## Performance Optimization

### 10.1 Database Optimization

**Query Optimization:**
```sql
-- Add indexes for common queries
CREATE INDEX CONCURRENTLY idx_audit_logs_admin_action 
ON audit_logs(admin_id, action, created_at);

CREATE INDEX CONCURRENTLY idx_sessions_active 
ON admin_sessions(admin_id, expires_at) 
WHERE expires_at > NOW();

-- Partition large tables
CREATE TABLE audit_logs_2024 PARTITION OF audit_logs
FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');
```

**Connection Pooling:**
```javascript
// config/database.js
const { Pool } = require('pg');

const pool = new Pool({
  host: process.env.DB_HOST,
  port: process.env.DB_PORT,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  max: 20,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});
```

### 10.2 Caching Strategy

**Redis Caching:**
```javascript
// services/cache.js
const redis = require('redis');
const client = redis.createClient(process.env.REDIS_URL);

class CacheService {
  async get(key) {
    const value = await client.get(key);
    return value ? JSON.parse(value) : null;
  }
  
  async set(key, value, ttl = 3600) {
    await client.setex(key, ttl, JSON.stringify(value));
  }
  
  async invalidate(pattern) {
    const keys = await client.keys(pattern);
    if (keys.length > 0) {
      await client.del(keys);
    }
  }
}
```

### 10.3 Frontend Optimization

**Build Optimization:**
```javascript
// vite.config.js
export default {
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          charts: ['chart.js', 'react-chartjs-2'],
          utils: ['lodash', 'date-fns']
        }
      }
    },
    chunkSizeWarningLimit: 1000
  }
};
```

## Disaster Recovery

### 11.1 Recovery Time Objectives

**RTO/RPO Targets:**
- **Critical Systems**: RTO 1 hour, RPO 15 minutes
- **Standard Systems**: RTO 4 hours, RPO 1 hour
- **Non-Critical Systems**: RTO 24 hours, RPO 4 hours

### 11.2 Disaster Recovery Plan

**Multi-Region Setup:**
```bash
# Primary region: us-east-1
# DR region: us-west-2

# Set up cross-region replication
aws s3api put-bucket-replication \
  --bucket harbotlist-admin-frontend \
  --replication-configuration file://replication.json

# Set up RDS cross-region backup
aws rds create-db-snapshot \
  --db-instance-identifier admin-db \
  --db-snapshot-identifier admin-db-dr-snapshot

aws rds copy-db-snapshot \
  --source-db-snapshot-identifier admin-db-dr-snapshot \
  --target-db-snapshot-identifier admin-db-dr-snapshot-west \
  --source-region us-east-1 \
  --target-region us-west-2
```

**Failover Procedures:**
```bash
#!/bin/bash
# failover-to-dr.sh

# Update Route 53 to point to DR region
aws route53 change-resource-record-sets \
  --hosted-zone-id Z123456789 \
  --change-batch file://failover-changeset.json

# Start DR instances
aws ec2 start-instances \
  --instance-ids i-1234567890abcdef0 \
  --region us-west-2

# Restore database from latest backup
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier admin-db-dr \
  --db-snapshot-identifier admin-db-dr-snapshot-west \
  --region us-west-2

# Update application configuration
sed -i 's/us-east-1/us-west-2/g' /opt/admin-dashboard/.env

# Start services
sudo systemctl start admin-dashboard
```

---

## Appendix A: Monitoring Dashboards

### CloudWatch Dashboard Configuration
```json
{
  "widgets": [
    {
      "type": "metric",
      "properties": {
        "metrics": [
          ["HarborList/Admin", "LoginAttempts"],
          [".", "SuccessfulLogins"],
          [".", "FailedLogins"]
        ],
        "period": 300,
        "stat": "Sum",
        "region": "us-east-1",
        "title": "Authentication Metrics"
      }
    },
    {
      "type": "metric",
      "properties": {
        "metrics": [
          ["AWS/ApplicationELB", "ResponseTime", "LoadBalancer", "admin-alb"],
          [".", "RequestCount", ".", "."],
          [".", "HTTPCode_ELB_5XX_Count", ".", "."]
        ],
        "period": 300,
        "stat": "Average",
        "region": "us-east-1",
        "title": "Application Performance"
      }
    }
  ]
}
```

## Appendix B: Automation Scripts

### Deployment Automation
```yaml
# .github/workflows/deploy.yml
name: Deploy Admin Dashboard

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '18'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run tests
        run: npm test
        
      - name: Build application
        run: npm run build
        
      - name: Deploy to AWS
        run: |
          aws s3 sync dist/ s3://harbotlist-admin-frontend/
          aws cloudfront create-invalidation --distribution-id ${{ secrets.CLOUDFRONT_ID }} --paths "/*"
```

---

*Last Updated: September 28, 2024*
*Version: 1.0*