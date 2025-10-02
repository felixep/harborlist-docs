# Backup and Recovery Procedures

## Overview

This document outlines the backup and recovery procedures for the HarborList platform to ensure data protection and business continuity.

## Backup Strategy

### Data Classification

#### Critical Data
- User accounts and profiles
- Boat listings and metadata
- Transaction records
- System configuration

#### Important Data
- User activity logs
- Search analytics
- Media files and images
- Application logs

#### Non-Critical Data
- Temporary files
- Cache data
- Development artifacts

### Backup Types

#### Full Backups
- **Frequency:** Weekly
- **Retention:** 4 weeks
- **Scope:** Complete database and file system
- **Storage:** AWS S3 with cross-region replication

#### Incremental Backups
- **Frequency:** Daily
- **Retention:** 7 days
- **Scope:** Changed data since last backup
- **Storage:** AWS S3 with versioning

#### Transaction Log Backups
- **Frequency:** Every 15 minutes
- **Retention:** 24 hours
- **Scope:** Database transaction logs
- **Storage:** Local and S3

## Backup Procedures

### Database Backups

#### Automated Backups
```bash
# RDS automated backups
aws rds create-db-snapshot \
  --db-instance-identifier harborlist-prod \
  --db-snapshot-identifier harborlist-backup-$(date +%Y%m%d-%H%M%S)
```

#### Manual Backup Process
1. **Pre-backup validation**
   - Check database connectivity
   - Verify backup storage availability
   - Confirm backup window

2. **Execute backup**
   - Create database snapshot
   - Export critical tables
   - Validate backup integrity

3. **Post-backup verification**
   - Verify backup completion
   - Test backup file integrity
   - Update backup logs

### File System Backups

#### Media Files
```bash
# Sync media files to backup bucket
aws s3 sync s3://harborlist-media s3://harborlist-media-backup \
  --delete --storage-class GLACIER
```

#### Configuration Files
```bash
# Backup configuration
tar -czf config-backup-$(date +%Y%m%d).tar.gz \
  /opt/harborlist/config/
```

### Application State Backups

#### Environment Configuration
- Environment variables
- SSL certificates
- API keys and secrets
- Infrastructure configuration

#### Code and Deployments
- Application code snapshots
- Deployment artifacts
- Infrastructure as Code templates

## Recovery Procedures

### Recovery Time Objectives (RTO)

| Component | RTO Target | Maximum Downtime |
|-----------|------------|------------------|
| Database | 1 hour | 2 hours |
| Application | 30 minutes | 1 hour |
| Media Files | 2 hours | 4 hours |
| Full System | 4 hours | 8 hours |

### Recovery Point Objectives (RPO)

| Data Type | RPO Target | Maximum Data Loss |
|-----------|------------|-------------------|
| Critical Data | 15 minutes | 30 minutes |
| Important Data | 1 hour | 2 hours |
| Non-Critical | 24 hours | 48 hours |

### Database Recovery

#### Point-in-Time Recovery
```bash
# Restore database to specific point in time
aws rds restore-db-instance-to-point-in-time \
  --source-db-instance-identifier harborlist-prod \
  --target-db-instance-identifier harborlist-recovery \
  --restore-time 2024-01-01T12:00:00Z
```

#### Snapshot Recovery
```bash
# Restore from snapshot
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier harborlist-recovery \
  --db-snapshot-identifier harborlist-backup-20240101-120000
```

### Application Recovery

#### Infrastructure Recovery
1. **Deploy infrastructure**
   ```bash
   cd infrastructure
   npm run deploy:prod
   ```

2. **Restore configuration**
   ```bash
   # Restore environment variables
   aws ssm put-parameter --name "/harborlist/config" --value "$(cat backup-config.json)"
   ```

3. **Deploy application**
   ```bash
   # Deploy backend services
   cd backend && npm run deploy
   
   # Deploy frontend
   cd frontend && npm run deploy
   ```

#### Data Recovery
1. **Restore database**
2. **Restore media files**
3. **Validate data integrity**
4. **Update application configuration**

### File Recovery

#### Media File Recovery
```bash
# Restore media files from backup
aws s3 sync s3://harborlist-media-backup s3://harborlist-media \
  --delete
```

#### Configuration Recovery
```bash
# Extract configuration backup
tar -xzf config-backup-20240101.tar.gz -C /opt/harborlist/
```

## Disaster Recovery

### Disaster Scenarios

#### Data Center Outage
- **Impact:** Complete service unavailability
- **Recovery:** Failover to secondary region
- **RTO:** 4 hours
- **RPO:** 1 hour

#### Database Corruption
- **Impact:** Data integrity issues
- **Recovery:** Point-in-time restore
- **RTO:** 2 hours
- **RPO:** 15 minutes

#### Security Breach
- **Impact:** Potential data compromise
- **Recovery:** Secure restore from clean backup
- **RTO:** 8 hours
- **RPO:** 1 hour

### Disaster Recovery Plan

#### Phase 1: Assessment (0-30 minutes)
1. Identify disaster scope
2. Activate disaster recovery team
3. Assess data integrity
4. Determine recovery strategy

#### Phase 2: Recovery (30 minutes - 4 hours)
1. Execute recovery procedures
2. Restore critical systems first
3. Validate system functionality
4. Restore remaining services

#### Phase 3: Validation (4-6 hours)
1. Comprehensive system testing
2. Data integrity verification
3. Performance validation
4. Security assessment

#### Phase 4: Communication (Ongoing)
1. Stakeholder notifications
2. Status updates
3. Post-incident reporting
4. Lessons learned documentation

## Monitoring and Testing

### Backup Monitoring
- Automated backup success/failure alerts
- Backup file integrity checks
- Storage capacity monitoring
- Recovery time testing

### Recovery Testing
- **Frequency:** Monthly
- **Scope:** Partial recovery testing
- **Documentation:** Test results and improvements
- **Validation:** Recovery procedures verification

### Compliance and Auditing
- Backup retention compliance
- Recovery procedure documentation
- Audit trail maintenance
- Regulatory requirement adherence

## Emergency Contacts

### Internal Team
- **Operations Manager:** [Contact Info]
- **Database Administrator:** [Contact Info]
- **Security Team:** [Contact Info]
- **Development Lead:** [Contact Info]

### External Vendors
- **AWS Support:** [Support Case Process]
- **Database Vendor:** [Support Contact]
- **Security Consultant:** [Contact Info]

## Documentation and Training

### Procedure Documentation
- Step-by-step recovery guides
- Emergency contact lists
- System architecture diagrams
- Recovery decision trees

### Team Training
- Regular disaster recovery drills
- Procedure walkthrough sessions
- New team member onboarding
- Annual training updates

This backup and recovery plan is reviewed quarterly and updated as needed to ensure effectiveness and compliance with business requirements.
