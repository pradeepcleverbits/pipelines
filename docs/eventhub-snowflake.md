# Pipeline Documentation: Azure Event Hub to Snowflake

## Pipeline Overview
- **Pipeline Name**: `cleverbits_ross_eventhub-snowflake`
- **Version**: $GIT_TAG$
- **Created**: $GIT_CREATION_DATE$
- **Last Modified**: $GIT_LAST_MODIFIED$
- **Created By**: pradeep@cleverbits.io
- **Business Owner**: amardeep@cleverbits.io
- **Technical Owner**: terry@cleverbits.io

> Note: Dates and versions are automatically managed through Git history.
> - Creation date reflects the first commit of this file
> - Last modified date reflects the most recent commit
> - Version reflects the latest Git tag

## Business Context
- **Purpose**: [Describe the business purpose of this pipeline]
- **Source System**: Azure Event Hub [Include specific source details]
- **Target System**: Snowflake [Include specific target details]
- **Data Flow Type**: [Real-time/Batch]
- **SLA Requirements**: [If any]
- **Data Retention Requirements**: [Specify retention periods. For example, eventhub max retention of data is 24hrs in basic tier]

## Prerequisites
1. **Access Requirements**:
    - Azure Event Hub access
    - Snowflake account access
    - StreamSets Data Collector access

2. **Required Roles/Permissions**:
   ```
   Azure Event Hub:
   - Role: [specify role]
   - Permissions: [list required permissions]

   Snowflake:
   - Role: [specify role]
   - Permissions: [list required permissions]
   ```

## Pipeline Configuration

### Source: Azure Event Hub Consumer
```
Stage Type: Azure Event Hub Consumer
Configuration:
- Event Hub Name: ${P_EVENTHUB_NAME}
- Consumer Group: ${P_CONSUMER_GROUP}
- Number of Threads: ${P_THREADS}
- Max Batch Size: ${P_BATCH_SIZE}
- Authentication Method: [Specify method]
```

### Processors
1. **Field Remover**
   ```
   Purpose: Remove unnecessary fields
   Fields to Remove:
   - field1
   - field2
   ```

2. **Field Type Converter**
   ```
   Purpose: Ensure correct data types
   Conversions:
   - timestamp: STRING to DATETIME
   - amount: STRING to DECIMAL(10,2)
   ```

### Destination: Snowflake
```
Stage Type: Snowflake
Configuration:
- Account: ${P_SF_ACCOUNT}
- Database: ${P_SF_DATABASE}
- Schema: ${P_SF_SCHEMA}
- Table: ${P_SF_TABLE}
- Stage Name: ${P_SF_STAGE}
- Authentication Method: [Specify method]
```

## Pipeline Parameters
| Parameter Name | Description | Default Value | Valid Values | Notes                                |
|---------------|-------------|---------------|--------------|--------------------------------------|
| P_EVENTHUB_NAME | Event Hub instance name | - | - | Production: myeventhub-prod          |
| P_CONSUMER_GROUP | Consumer group name | $Default | - | One consumer group per pipeline      |
| P_THREADS | Number of threads | 1 | 1-8 | Based on partition count             |
| P_BATCH_SIZE | Records per batch | 1000 | 100-10000 | Tune based on record size            |
| P_SF_ACCOUNT | Snowflake account | - | - | Format: org-account                  |
| P_SF_DATABASE | Target database | - | - | -                                    |
| P_SF_SCHEMA | Target schema | - | - | -                                    |
| P_SF_TABLE | Target table | - | - | -                                    |
| P_SF_STAGE | Stage name | - | - | Internal snowflake stage |

## Error Handling
1. **Pipeline Level**:
    - Error Records: Write to Event Hub error topic
    - Maximum Error Records: ${P_MAX_ERROR_RECORDS}
    - Error Record Policy: [Specify policy]

2. **Stage Level**:
   ```
   Azure Event Hub Consumer:
   - On Error: Send to Error
   - Retries: 0

   Snowflake:
   - On Error: Stop pipeline
   - Retry Count: 0
   ```

## Monitoring and Alerting
1. **Metrics to Monitor**:
    - Records/second
    - Batch processing time
    - Error records count
    - Pipeline status

2. **Alert Thresholds**:
   ```
   Critical Alerts:
   - Pipeline stopped: Immediate
   - Error records > 1000: 5 minutes
   - Processing delay > 10 minutes: 5 minutes

   Warning Alerts:
   - Batch processing time > 30 seconds
   - CPU usage > 80%
   - Memory usage > 80%
   ```

## Data Quality Checks
1. **Pre-Load Validation**:
    - Required fields presence
    - Data type compatibility
    - Value range checks

2. **Post-Load Validation**:
   ```sql
   -- Record count validation
   SELECT COUNT(*) FROM target_table WHERE ingestion_date = CURRENT_DATE;

   -- Duplicate check
   SELECT message_id, COUNT(*)
   FROM target_table
   GROUP BY message_id
   HAVING COUNT(*) > 1;
   ```

## Recovery Procedures
1. **Pipeline Failure**:
   ```
   1. Check error logs
   2. Review error records in error topic
   3. Fix root cause
   4. Adjust offset if needed
   5. Restart pipeline
   ```

2. **Data Recovery**:
   ```
   1. Identify missing data period
   2. Update start offset
   3. Create recovery pipeline if needed
   4. Validate recovered data
   ```

## Performance Tuning
1. **Pipeline Configurations**:
   ```
   Optimal Settings:
   - Batch Size: 1000
   - Number of Threads: 4
   - Memory Limit: 512 MB
   ```

2. **Known Bottlenecks**:
    - List known performance bottlenecks
    - Document solutions/workarounds

## Change Log
| Date | Version | Changed By | Description |
|------|---------|------------|-------------|
| YYYY-MM-DD | 1.0 | [Name] | Initial version |

## Supporting Documentation
1. **Related Documents**:
    - Link to design documents
    - Link to test cases
    - Link to runbook

2. **Contact Information**:
   ```
   Support Team: [team name]
   Email: [email]
   Slack Channel: [channel]
   On-Call: [contact details]
   ```