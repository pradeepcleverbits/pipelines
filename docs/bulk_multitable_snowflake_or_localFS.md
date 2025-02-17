# Pipeline Documentation: JDBC Multi-table Consumer to Local FS/Snowflake

## Pipeline Overview
- **Pipeline Name**: `cleverbits_jdbc-localfs-snowflake`
- **Version**: $GIT_TAG$
- **Created**: $GIT_CREATION_DATE$
- **Last Modified**: $GIT_LAST_MODIFIED$
- **Created By**: [REMOVED]
- **Business Owner**: [REMOVED]
- **Technical Owner**: [REMOVED]

> Note: Dates and versions are automatically managed through Git history.
> - Creation date reflects the first commit of this file
> - Last modified date reflects the most recent commit
> - Version reflects the latest Git tag

## Business Context
- **Purpose**: [Describe the business purpose of this pipeline]
- **Source System**: JDBC Database (Multi-table Consumer)
- **Target Systems**:
    - Local File System
    - Snowflake
- **Data Flow Type**: Batch
- **SLA Requirements**: [If any]
- **Data Retention Requirements**: [Specify retention periods]

## Prerequisites
1. **Access Requirements**:
    - Source database access
    - Local file system write access
    - Snowflake account access
    - StreamSets Data Collector access

2. **Required Roles/Permissions**:
   ```
   Source Database:
   - Read access to source tables
   - Transaction isolation level: SERIALIZABLE
   - Required database roles: [specify roles]

   Local File System:
   - Write permissions on target directory
   - Execute permissions on parent directories

   Snowflake:
   - Write access to target table and stage
   - Required warehouse privileges
   ```

## Security Notes
- All credentials should be stored in a secure credential store
- Database connections should use encrypted communication
- File system permissions should be properly configured
- Enable audit logging for all components
- Regular credential rotation

## Pipeline Configuration

### Source: JDBC Multi-table Consumer
```
Stage Type: JDBC Multi-table Consumer
Configuration:
- Connection String: ${P_JDBCQUERYCONSUMER_CONNECTION}
- Table Pattern: ${P_SOURCE_TABLE}
- Transaction Isolation: ${P_TRANSACTION_ISOLATION}
```

### Stream Selector
```
Condition 1: ${P_LOAD_TYPE} == 'FILE_BL'
  → Local File System Destination
Condition 2: ${P_LOAD_TYPE} == 'SNOW_BL'
  → Snowflake Destination
```

### Destination 1: Local File System
```
Stage Type: Local FS
Configuration:
- Directory Path: ${P_LOCAL_FS_DIR_PATH}
- Purge Source Files: ${P_PURGE_DATAFILE_FLAG}
```

### Destination 2: Snowflake
```
Stage Type: Snowflake
Configuration:
- Connection: ${P_SNOWFLAKE_CONNECTION}
- Stage Database: ${P_SNOWFLAKE_STAGE_DB}
- Stage Schema: ${P_SNOWFLAKE_STAGE_SCHEMA}
- Stage Name: ${P_SNOWFLAKE_STAGE_NAME}
```

## Pipeline Parameters
| Parameter Name | Description | Example Value | Notes |
|---------------|-------------|---------------|-------|
| P_JDBCQUERYCONSUMER_CONNECTION | JDBC connection ID | [SECURED] | Store in credential store |
| P_LOAD_TYPE | Destination selector | FILE_BL/SNOW_BL | Controls data flow path |
| P_LOCAL_FS_DIR_PATH | Target directory path | /home/sdc/target | Absolute path required |
| P_PURGE_DATAFILE_FLAG | Delete source after processing | true | Boolean value |
| P_SNOWFLAKE_CONNECTION | Snowflake connection ID | [SECURED] | Store in credential store |
| P_SNOWFLAKE_STAGE_DB | Stage database | CLEVERBITS | - |
| P_SNOWFLAKE_STAGE_NAME | Internal stage name | EMPLOYEES__FULL_LOAD__SNOWFLAKE_STAGE | - |
| P_SNOWFLAKE_STAGE_SCHEMA | Stage schema | CDC | - |
| P_SOURCE_TABLE | Source table name | EMPLOYEES | Supports patterns |
| P_TRANSACTION_ISOLATION | Transaction isolation level | TRANSACTION_SERIALIZABLE | Database specific |

## Error Handling
1. **Pipeline Level**:
    - Error Records: Write to error directory
    - Error Record Policy: [Specify policy]

2. **Stage Level**:
   ```
   JDBC Source:
   - On Error: Send to Error
   - Retries: 3
   - Transaction Timeout: 30 seconds

   Local FS:
   - On Error: Stop Pipeline
   - Retries: 3

   Snowflake:
   - On Error: Stop Pipeline
   - Retry Count: 3
   ```

## Monitoring and Alerting
1. **Metrics to Monitor**:
    - Records processed/minute
    - Batch processing time
    - Transaction duration
    - File write performance
    - Snowflake load status
    - Error records count
    - Pipeline status

2. **Alert Thresholds**:
   ```
   Critical Alerts:
   - Pipeline stopped: Immediate
   - Error records > 1000: 5 minutes
   - Transaction timeout: Immediate
   - Directory space > 80%: 15 minutes

   Warning Alerts:
   - Batch processing time > 5 minutes
   - CPU usage > 80%
   - Memory usage > 80%
   ```

## Data Quality Checks
1. **Pre-Load Validation**:
    - Source table accessibility
    - Required fields presence
    - Transaction isolation level
    - Available disk space

2. **Post-Load Validation**:
   ```sql
   -- Record count validation
   SELECT COUNT(*) FROM target_table WHERE load_date = CURRENT_DATE;

   -- Compare source and target counts
   SELECT 
     (SELECT COUNT(*) FROM source_table) as source_count,
     (SELECT COUNT(*) FROM target_table WHERE batch_id = current_batch) as target_count;
   ```

## Recovery Procedures
1. **Pipeline Failure**:
   ```
   1. Check error logs
   2. Verify transaction state
   3. Check file system state
   4. Fix root cause
   5. Restart pipeline
   ```

2. **Data Recovery**:
   ```
   1. Identify last successful batch
   2. Verify source data availability
   3. Reset pipeline offset if needed
   4. Reprocess failed batch
   5. Validate recovered data
   ```

## Performance Tuning
1. **Pipeline Configurations**:
   ```
   Optimal Settings:
   - Batch Size: 1000 records
   - Transaction Timeout: 30 seconds
   - Memory Limit: 512 MB
   - File Write Buffer: 8192 bytes
   ```

2. **Known Bottlenecks**:
    - Database read performance
    - Transaction locks
    - Disk I/O for file writes
    - Network latency to Snowflake
    - Available memory for sorting/processing

## Supporting Documentation
1. **Related Documents**:
    - Link to design documents
    - Link to test cases
    - Link to runbook

2. **Contact Information**:
   ```
   Support Team: [REMOVED]
   Email: [REMOVED]
   Slack Channel: [REMOVED]
   On-Call: [REMOVED]
   ```