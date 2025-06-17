# Metabase

## Overview

Use the Metabase API. This package provides a comprehensive set of activities for interacting with Metabase business intelligence and analytics platform, including question management, metrics retrieval, database operations, and alert monitoring.

## Components

### Activities

#### Questions Management

- **Get Question** (`metabase.questions.getQuestionById`): Retrieve a specific question by its ID
- **Get All Questions** (`metabase.questions.getAllQuestions`): Retrieve multiple questions
- **Get Question Result** (`metabase.questions.getQuestionResult`): Retrieve the result of a question in a specific format

#### Metrics Management

- **Get Metric** (`metabase.metrics.getMetricById`): Retrieve a specific metric by its ID
- **Get All Metrics** (`metabase.metrics.getAllMetrics`): Retrieve multiple metrics

#### Database Operations

- **Add Database** (`metabase.databases.addDatabase`): Add a new datasource to the metabase instance
- **Get All Databases** (`metabase.databases.getAllDatabases`): Retrieve multiple databases
- **Get Database Fields** (`metabase.databases.getDatabaseFields`): Retrieve fields from a specific database

#### Alerts Management

- **Get Alert** (`metabase.alerts.getAlert`): Retrieve a specific alert by its ID
- **Get All Alerts** (`metabase.alerts.getAllAlerts`): Retrieve multiple alerts

## Dependencies

### Required

None - this package has no required dependencies.

### Optional

None - this package currently has no optional dependencies.

## Usage

### Questions and Data Retrieval

```yaml
# Get a specific question
activity: metabase.questions.getQuestionById
inputs:
  questionId: "42"

# Get all questions
activity: metabase.questions.getAllQuestions

# Get question results in specific format
activity: metabase.questions.getQuestionResult
inputs:
  questionId: "42"
  format: "json"
```

### Metrics Management

```yaml
# Get a specific metric
activity: metabase.metrics.getMetricById
inputs:
  metricId: "15"

# Get all metrics
activity: metabase.metrics.getAllMetrics
```

### Database Configuration

```yaml
# Add a PostgreSQL database
activity: metabase.databases.addDatabase
inputs:
  engine: "postgres"
  host: "db.company.com"
  name: "Production Database"
  port: "5432"
  user: "metabase_user"
  password: "secure_password"
  dbName: "analytics"
  fullSync: "true"

# Add a MySQL database
activity: metabase.databases.addDatabase
inputs:
  engine: "mysql"
  host: "mysql.company.com:3306"
  name: "MySQL Analytics"
  port: "3306"
  user: "analytics_user"
  password: "mysql_password"
  dbName: "reporting"
  fullSync: "false"

# Get all configured databases
activity: metabase.databases.getAllDatabases

# Get database schema fields
activity: metabase.databases.getDatabaseFields
inputs:
  databaseId: "3"
```

### Alerts Monitoring

```yaml
# Get a specific alert
activity: metabase.alerts.getAlert
inputs:
  alertId: "7"

# Get all alerts
activity: metabase.alerts.getAllAlerts
```

### Complete Dashboard Setup Workflow

```yaml
# Workflow for setting up a new analytics environment
steps:
  - name: Add Production Database
    activity: metabase.databases.addDatabase
    inputs:
      engine: "postgres"
      host: "${DB_HOST}"
      name: "Production Analytics"
      port: "${DB_PORT}"
      user: "${DB_USER}"
      password: "${DB_PASSWORD}"
      dbName: "${DB_NAME}"
      fullSync: "true"

  - name: Verify Database Fields
    activity: metabase.databases.getDatabaseFields
    inputs:
      databaseId: "${DATABASE_ID}"

  - name: Check Existing Questions
    activity: metabase.questions.getAllQuestions

  - name: Monitor Alerts
    activity: metabase.alerts.getAllAlerts
```

## Configuration

### Required Configuration

Activities require Metabase API connectivity:

- **Metabase server URL**: Base URL of your Metabase instance
- **Authentication**: Valid API credentials or session tokens
- **Network access**: Connectivity to Metabase server

### Database Configuration

#### Supported Database Engines

- **PostgreSQL**: Default engine, widely supported
- **MySQL**: Popular relational database
- **SQLite**: File-based database for development
- **H2**: Embedded database for testing
- **BigQuery**: Google Cloud data warehouse
- **Snowflake**: Cloud data platform
- **Redshift**: Amazon data warehouse

#### Connection Parameters

- **host**: Database server hostname or IP address
- **port**: Database server port number
- **user/password**: Database authentication credentials
- **dbName**: Target database or schema name
- **filePath**: For file-based databases (SQLite, H2)
- **fullSync**: Enable complete metadata synchronization

### Data Format Options

#### Question Result Formats

- **CSV**: Comma-separated values for spreadsheet import
- **JSON**: Structured data for API integration
- **XLSX**: Excel format for business users

### Authentication

These activities use console handlers for demonstration and validation purposes. In a production environment, you would need:

- Metabase API authentication (username/password or API key)
- Proper user permissions for database management
- Network connectivity to Metabase server and target databases

## Metabase Integration Patterns

### Analytics Workflow

1. **Setup Databases**: Add and configure data sources
2. **Create Questions**: Build analytical queries and visualizations
3. **Define Metrics**: Establish key business metrics
4. **Configure Alerts**: Set up monitoring and notifications
5. **Export Results**: Extract data in various formats

### Multi-Environment Setup

```yaml
# Development environment
database:
  engine: "postgres"
  host: "dev-db.company.com"
  name: "Development Analytics"

# Staging environment
database:
  engine: "postgres"
  host: "staging-db.company.com"
  name: "Staging Analytics"

# Production environment
database:
  engine: "postgres"
  host: "prod-db.company.com"
  name: "Production Analytics"
```

### Data Pipeline Integration

```yaml
# ETL workflow with Metabase validation
steps:
  - name: Extract Data
    activity: external.etl.extract

  - name: Transform Data
    activity: external.etl.transform

  - name: Load Data
    activity: external.etl.load

  - name: Verify in Metabase
    activity: metabase.questions.getQuestionResult
    inputs:
      questionId: "data_quality_check"
      format: "json"

  - name: Check Data Alerts
    activity: metabase.alerts.getAllAlerts
```

## Best Practices

### Database Management

- Use service accounts for database connections
- Implement proper security and access controls
- Regular backup and maintenance of Metabase metadata
- Monitor database performance and query efficiency

### Question and Dashboard Organization

- Use clear naming conventions for questions and dashboards
- Organize content in logical collections
- Document question purposes and data sources
- Regular review and cleanup of unused content

### Performance Optimization

- Enable database connection pooling
- Configure appropriate query timeouts
- Use caching for frequently accessed data
- Monitor and optimize expensive queries

### Security Considerations

- Secure database credentials using secrets management
- Implement row-level security where needed
- Regular audit of user permissions and access
- Use HTTPS for all Metabase communications

## Data Export and Integration

### Automated Reporting

```yaml
# Daily report generation
schedule: "0 8 * * *" # 8 AM daily
activity: metabase.questions.getQuestionResult
inputs:
  questionId: "daily_sales_report"
  format: "csv"
# Output can be sent to email, S3, etc.
```

### API Integration

```yaml
# Real-time metrics for external systems
activity: metabase.questions.getQuestionResult
inputs:
  questionId: "current_kpis"
  format: "json"
# JSON output can be consumed by dashboards, APIs, etc.
```

### Business Intelligence Workflow

```yaml
# Complete BI pipeline
steps:
  - name: Refresh Data Sources
    activity: metabase.databases.getAllDatabases

  - name: Generate Executive Report
    activity: metabase.questions.getQuestionResult
    inputs:
      questionId: "executive_dashboard"
      format: "xlsx"

  - name: Check Alert Status
    activity: metabase.alerts.getAllAlerts

  - name: Export Key Metrics
    activity: metabase.metrics.getAllMetrics
```

## Troubleshooting

### Common Issues

- **Database connection failures**: Verify network connectivity and credentials
- **Query timeouts**: Optimize queries or increase timeout settings
- **Permission errors**: Check user access rights in Metabase
- **Format issues**: Ensure requested export format is supported

### Error Handling

- Implement retry mechanisms for transient failures
- Log detailed error information for debugging
- Provide meaningful error messages for business users
- Establish escalation procedures for data quality issues

### Monitoring and Maintenance

- Regular health checks of database connections
- Monitor Metabase server performance and capacity
- Automated alerting for failed data refreshes
- Periodic review of question and dashboard usage

## Notes

- All activities in this package use console handlers for output and validation
- These activities provide structure and validation for Metabase API operations
- For production use, consider implementing corresponding PipelineRef handlers with actual Metabase API integration
- Ensure proper Metabase server access and database connectivity before using these activities
- Consider implementing data governance and quality controls for business-critical analytics
- Regular backup of Metabase application database is recommended for disaster recovery
