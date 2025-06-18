# Mixed Operations

## Overview

The Mixed Operations package contains a diverse collection of activities spanning multiple technology domains and platforms. This package serves as a utility collection for various operational tasks including scripting, notifications, cloud services, database operations, and employee onboarding workflows.

## Components

### Activities

This package provides 20 different activities covering:

- **Scripting Operations**: Python and Bash script execution
- **Notification Systems**: Gmail, Slack, and Teams notifications
- **Cloud Platforms**: Azure, Google Cloud BigQuery operations
- **Employee Onboarding**: Microsoft 365, Azure AD, Zoho, and other HR systems
- **Development Tools**: Azure DevOps, SonarQube, Jira, Visual Studio access management
- **Data Operations**: PostgreSQL data extraction, Spark ETL processing
- **Collaboration Tools**: Trello, LMS setup
- **Workflow Management**: Manual approval processes

## Dependencies

### Required

None - This package operates independently

### Optional

None - Activities are designed to be self-contained

## Usage

Each activity in this package is designed for console output and can be used individually based on your specific operational needs. Activities include proper input validation and descriptive output messages.

### Example Usage Patterns

**Script Execution**:

- Use Python or Bash script activities for automated task execution
- Suitable for data processing, system administration, and automation

**Notification Workflows**:

- Integrate Gmail, Slack, or Teams activities into larger workflows
- Perfect for status updates and alert systems

**Employee Onboarding**:

- Chain multiple employee onboarding activities for complete user setup
- Covers identity management, access provisioning, and tool configuration

**Development Operations**:

- Use development tool activities for project setup and access management
- Includes version control, CI/CD, and code quality tools

## Configuration

Each activity includes:

- Input validation with clear parameter descriptions
- Console-based execution with informative output
- Template expressions for dynamic content
- Proper error handling and status reporting

## Activities List

1. **python-script** - Execute Python scripts
2. **bash-script** - Execute Bash scripts
3. **gmail-notification** - Send Gmail notifications
4. **slack-notification** - Send Slack notifications
5. **spark-etl** - Process data with Spark ETL
6. **bigquery-ingest** - Ingest data into BigQuery
7. **m365-create-account** - Create Microsoft 365 accounts
8. **azure-ad-employee** - Add employees to Azure AD
9. **zoho-employee-profile** - Create Zoho employee profiles
10. **teams-invite** - Send Teams channel invitations
11. **slack-create-account** - Create Slack accounts
12. **azure-devops-access** - Provide Azure DevOps access
13. **azure-resource-access** - Configure Azure resource access
14. **sonarqube-access** - Provide SonarQube access
15. **jira-create-account** - Create Jira accounts
16. **postgresql-extract** - Extract PostgreSQL data
17. **trello-add-user** - Add users to Trello
18. **visual-studio-access** - Provide Visual Studio access
19. **manual-approval** - Manual approval workflows
20. **lms-setup** - Learning Management System setup

## Best Practices

- Review input requirements before executing activities
- Use template expressions for dynamic parameter values
- Chain activities using workflow orchestration for complex processes
- Monitor console output for execution status and results

## Troubleshooting

### Common Issues

- **Missing Parameters**: Ensure all required inputs are provided
- **Template Errors**: Verify template expression syntax
- **Access Issues**: Check permissions for external system integrations

### Support

For technical support and questions:

- Review activity documentation for specific requirements
- Check ClrSlate platform logs for execution details
- Contact Team ClrSlate for package-specific issues
