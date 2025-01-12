# Pipeline Documentation: Azure Data Lake Storage to Snowflake
# Provided below is the template format. Fill in or change value(s) according to your business use case.

## Pipeline Overview
- **Pipeline Name**: `cleverbits_adls-snowflake`
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
- **Source System**: Azure Data Lake Storage (ADLS)
- **Target System**: Snowflake
- **Data Flow Type**: Batch
- **SLA Requirements**: [If any]
- **Data Retention Requirements**: [Specify retention periods]

## Prerequisites
1. **Access Requirements**:
    - Azure Data Lake Storage access
    - Snowflake account access
    - StreamSets Data Collector access

2. **Required Roles/Permissions**:
   ```
   Azure Data Lake Storage: 
   - Auth method(Shared Key, Azure Managed Identity or Oauth
   with Service principal): 
   - Fields related to Auth method(Shared key/Application ID/Tenant ID): 
   - Permissions: Storage Blob Data Reader

   Snowflake:
   - Auth method[User credentials/Key pair path/key pair content/Oauth]:
   - Fields related to Auth method:
   - Permissions: Write access to target table and stage
   ```

## Security Notes
- All credentials should be stored in a secure credential store
- Connection strings should be encrypted
- Access keys should be rotated regularly
- Use managed identities where possible
- Enable audit logging for all components

## Pipeline Configuration

### Source: Azure Data Lake Storage
```
Stage Type: Azure Data Lake Storage
Configuration:
- Account FQDN: ${P_AZURE_ACCOUNT_FQDN}
- Container: ${P_STORAGE_CONTAINER}
- Data Path: ${P_DATA_PATH_IN_CONTAINER}
- File Pattern: ${P_DATA_FILE_PATTERN}
- Number of Threads: ${P_NUMBER_OF_THREADS_TO_READ_FROM_ADLS}
- Data Format: ${P_SOURCE_DATA_FORMAT}
- Authentication Method: Storage Account Key
```

### Destination: Snowflake
```
Stage Type: Snowflake
Configuration:
- Account: ${P_SNOWFLAKE_ACCOUNT}
- Organization: ${P_SNOWFLAKE_ORGANIZATION}
- Warehouse: ${P_SNOWFLAKE_TARGET_WAREHOUSE}
- Database: ${P_SNOWFLAKE_TARGET_DB}
- Schema: ${P_SNOWFLAKE_TARGET_SCHEMA}
- Table: ${P_SNOWFLAKE_TARGET_TABLE}
- Stage Name: ${P_SNOWFLAKE_STAGE_NAME}
- Stage Database: ${P_SNOWFLAKE_STAGE_DB}
- Stage Schema: ${P_SNOWFLAKE_STAGE_SCHEMA}
- Authentication: User & Password
```

## Pipeline Parameters
| Parameter Name | Description | Example Value | Notes |
|---------------|-------------|---------------|-------|
| P_AZURE_ACCOUNT_FQDN | ADLS account FQDN | *.blob.core.windows.net | - |
| P_DATA_FILE_PATTERN | File pattern to process | sdc* | Supports wildcards |
| P_DATA_PATH_IN_CONTAINER | Data path within container | data/input | - |
| P_NUMBER_OF_THREADS_TO_READ_FROM_ADLS | Number of reader threads | 1 | Tune based on performance |
| P_STORAGE_CONTAINER | ADLS container name | input-data | - |
| P_SOURCE_DATA_FORMAT | Source data format | JSON | Supported: JSON, CSV, AVRO |
| P_SNOWFLAKE_ACCOUNT | Snowflake account | [SECURED] | Store in credential store |
| P_SNOWFLAKE_ORGANIZATION | Snowflake org | [SECURED] | Store in credential store |
| P_SNOWFLAKE_TARGET_DB | Target database | PROD_DB | - |
| P_SNOWFLAKE_TARGET_SCHEMA | Target schema | PROD_SCHEMA | - |
| P_SNOWFLAKE_TARGET_TABLE | Target table | DESTINATION_TABLE | - |
| P_SNOWFLAKE_TARGET_WAREHOUSE | Compute warehouse | COMPUTE_WH | - |
| P_SNOWFLAKE_STAGE_NAME | Internal stage name | INTERNAL_STAGE | - |
| P_SNOWFLAKE_STAGE_DB | Stage database | STAGE_DB | - |
| P_SNOWFLAKE_STAGE_SCHEMA | Stage schema | STAGE_SCHEMA | - |
| P_EVENTHUB_NAMESPACE | Error handling namespace | dev-eventhub | For error records |
| P_ERROR_RECORD_EVENTHUB | Error records topic | error-records | - |
| P_SNOWFLAKE_USER | Snowflake username | [SECURED] | Store in credential store |
| P_SNOWFLAKE_PASSWORD | Snowflake password | [SECURED] | Store in credential store |
| P_STORAGE_ACCOUNT_KEY | ADLS account key | [SECURED] | Store in credential store |
| P_STORAGE_ACCOUNT_CONNECTION_STRING | ADLS connection string | [SECURED] | Store in credential store |

## Error Handling
1. **Pipeline Level**:
    - Error Records: Write to Event Hub topic ${P_ERROR_RECORD_EVENTHUB}
    - Error Record Policy: [Specify policy]

2. **Stage Level**:
   ```
   ADLS Source:
   - On Error: Send to Error
   - Retries: 3

   Snowflake:
   - On Error: Stop pipeline
   - Retry Count: 3
   ```

## Monitoring and Alerting
1. **Metrics to Monitor**:
    - Files processed/hour
    - Records processed/minute
    - Error records count
    - Pipeline status
    - Snowflake load status

2. **Alert Thresholds**:
   ```
   Critical Alerts:
   - Pipeline stopped: Immediate
   - Error records > 1000: 5 minutes
   - File processing delay > 30 minutes: 5 minutes

   Warning Alerts:
   - Batch processing time > 5 minutes
   - CPU usage > 80%
   - Memory usage > 80%
   ```

## Data Quality Checks
1. **Pre-Load Validation**:
    - File format validation
    - Required fields presence
    - Data type compatibility

2. **Post-Load Validation**:
   ```sql
   -- Record count validation
   SELECT COUNT(*) FROM target_table WHERE load_date = CURRENT_DATE;

   -- Duplicate check
   SELECT id, COUNT(*)
   FROM target_table
   GROUP BY id
   HAVING COUNT(*) > 1;
   ```

## Recovery Procedures
1. **Pipeline Failure**:
   ```
   1. Check error logs in Event Hub
   2. Review error records
   3. Fix root cause
   4. Reprocess failed files
   5. Restart pipeline
   ```

2. **Data Recovery**:
   ```
   1. Identify missing files
   2. Verify source files availability
   3. Create recovery pipeline if needed
   4. Validate recovered data
   ```

## Performance Tuning
1. **Pipeline Configurations**:
   ```
   Optimal Settings:
   - Number of Threads: Start with 1, increase based on performance
   - Memory Limit: 512 MB
   - Warehouse Size: XSMALL for < 1M records/hour
   ```

2. **Known Bottlenecks**:
    - Large JSON files parsing
    - Network bandwidth between ADLS and SDC
    - Snowflake compute resources

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