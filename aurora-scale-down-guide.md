# Aurora Instance Scale Down Guide

## Overview
Scale down Aurora cluster instances from memory optimized (db.r7g.xlarge) to burstable (db.t4g.medium).

**Current State:** Writer and Reader instances on db.r7g.xlarge  
**Target State:** Both instances on db.t4g.medium

## Prerequisites
- Aurora cluster with writer and reader instances
- Maintenance window scheduled (recommended)
- Application can handle brief connectivity interruption
- Monitoring tools ready to track performance post-change

## Step-by-Step Process

### 1. Pre-Change Validation
```bash
# Check current instance classes
aws rds describe-db-instances --query 'DBInstances[?contains(DBInstanceIdentifier, `your-cluster-name`)].{Instance:DBInstanceIdentifier,Class:DBInstanceClass,Status:DBInstanceStatus}'
```

### 2. Scale Down Reader Instance First
```bash
# Modify reader instance
aws rds modify-db-instance \
    --db-instance-identifier your-cluster-name-reader \
    --db-instance-class db.t4g.medium \
    --apply-immediately
```

### 3. Monitor Reader Instance
```bash
# Check modification status
aws rds describe-db-instances \
    --db-instance-identifier your-cluster-name-reader \
    --query 'DBInstances[0].{Status:DBInstanceStatus,Class:DBInstanceClass,PendingClass:PendingModifiedValues.DBInstanceClass}'
```

### 4. Wait for Reader Completion
- Monitor until status returns to "available"
- Verify performance metrics are acceptable
- Typical downtime: 2-5 minutes

### 5. Scale Down Writer Instance
```bash
# Modify writer instance
aws rds modify-db-instance \
    --db-instance-identifier your-cluster-name-writer \
    --db-instance-class db.t4g.medium \
    --apply-immediately
```

### 6. Monitor Writer Instance
```bash
# Check modification status
aws rds describe-db-instances \
    --db-instance-identifier your-cluster-name-writer \
    --query 'DBInstances[0].{Status:DBInstanceStatus,Class:DBInstanceClass,PendingClass:PendingModifiedValues.DBInstanceClass}'
```

### 7. Post-Change Validation
```bash
# Verify both instances are updated
aws rds describe-db-instances --query 'DBInstances[?contains(DBInstanceIdentifier, `your-cluster-name`)].{Instance:DBInstanceIdentifier,Class:DBInstanceClass,Status:DBInstanceStatus}'
```

## Important Considerations

### Performance Impact
- **CPU Credits:** t4g instances use burstable CPU with credit system
- **Memory:** Reduced from 32 GiB to 4 GiB per instance
- **Network:** Lower baseline network performance

### Monitoring Post-Change
- Watch CPU credit balance
- Monitor query performance
- Check connection pool behavior
- Verify application response times

### Rollback Plan
If performance issues occur:
```bash
# Scale back up (reverse process)
aws rds modify-db-instance \
    --db-instance-identifier your-instance-name \
    --db-instance-class db.r7g.xlarge \
    --apply-immediately
```

## Timeline
- Reader modification: ~3-5 minutes
- Writer modification: ~3-5 minutes
- Total process: ~10-15 minutes

## Cost Impact
- Significant cost reduction (~75% savings)
- Monitor for potential performance degradation requiring scale-up