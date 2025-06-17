# SonarQube

## Overview

Building blocks for integrating SonarQube in CI/CD pipelines. This package provides a comprehensive set of activities for code quality analysis, including project management, scan execution, status monitoring, and quality gate validation within SonarQube environments.

## Components

### Activities

- **Start SonarQube Scan** (`sonarqube.scan.startScan`): Trigger a SonarQube scan for a project
- **Check Scan Status** (`sonarqube.scan.checkStatus`): Check the status of an ongoing SonarQube scan
- **Check Quality Gate** (`sonarqube.qualityGate.checkQualityGate`): Verify if the SonarQube project passes the quality gate
- **Create SonarQube Project** (`sonarqube.project.createProject`): Create a new project in SonarQube
- **Delete SonarQube Project** (`sonarqube.project.deleteProject`): Delete a project from SonarQube

## Dependencies

### Required

None - this package has no required dependencies.

### Optional

None - this package currently has no optional dependencies.

## Usage

### Project Management

```yaml
# Create a new SonarQube project
activity: sonarqube.project.createProject
inputs:
  projectKey: "my-app"
  projectName: "My Application"
  sonarHostUrl: "https://sonarqube.company.com"
  authToken: "squ_abc123xyz789"

# Delete a SonarQube project
activity: sonarqube.project.deleteProject
inputs:
  projectKey: "my-app"
  sonarHostUrl: "https://sonarqube.company.com"
  authToken: "squ_abc123xyz789"
```

### Code Scanning Workflow

```yaml
# Start a code scan
activity: sonarqube.scan.startScan
inputs:
  projectKey: "my-app"
  sourceDirectory: "./src"
  sonarHostUrl: "https://sonarqube.company.com"
  authToken: "squ_abc123xyz789"

# Check scan progress
activity: sonarqube.scan.checkStatus
inputs:
  projectKey: "my-app"
  sonarHostUrl: "https://sonarqube.company.com"
  authToken: "squ_abc123xyz789"

# Validate quality gate
activity: sonarqube.qualityGate.checkQualityGate
inputs:
  projectKey: "my-app"
  sonarHostUrl: "https://sonarqube.company.com"
  authToken: "squ_abc123xyz789"
```

### CI/CD Pipeline Integration

```yaml
# Complete pipeline workflow
steps:
  - name: Start Code Analysis
    activity: sonarqube.scan.startScan
    inputs:
      projectKey: "my-app-${BUILD_NUMBER}"
      sourceDirectory: "./src"
      sonarHostUrl: "${SONAR_HOST_URL}"
      authToken: "${SONAR_AUTH_TOKEN}"

  - name: Wait for Analysis
    activity: sonarqube.scan.checkStatus
    inputs:
      projectKey: "my-app-${BUILD_NUMBER}"
      sonarHostUrl: "${SONAR_HOST_URL}"
      authToken: "${SONAR_AUTH_TOKEN}"

  - name: Validate Quality Gate
    activity: sonarqube.qualityGate.checkQualityGate
    inputs:
      projectKey: "my-app-${BUILD_NUMBER}"
      sonarHostUrl: "${SONAR_HOST_URL}"
      authToken: "${SONAR_AUTH_TOKEN}"
```

## Configuration

### Required Configuration

All activities require basic SonarQube connectivity:

- **sonarHostUrl**: SonarQube server URL (e.g., "https://sonarqube.company.com")
- **authToken**: Valid SonarQube authentication token
- **projectKey**: Unique identifier for the SonarQube project

### Authentication

These activities use console handlers for demonstration and validation purposes. In a production environment, you would need:

- SonarQube authentication token with appropriate permissions
- Network connectivity to the SonarQube server
- Proper project permissions for scan execution and management

### Activity-Specific Configuration

#### Project Management

- **projectKey**: Must be unique across the SonarQube instance
- **projectName**: Human-readable project name for identification

#### Code Scanning

- **sourceDirectory**: Path to source code directory for analysis
- **projectKey**: Must reference an existing SonarQube project

#### Quality Gate Validation

- **projectKey**: Must reference a project with completed analysis
- Quality gate rules must be configured in SonarQube

## SonarQube Integration Patterns

### Basic Workflow

1. **Create Project**: Set up project in SonarQube
2. **Start Scan**: Initiate code analysis
3. **Monitor Progress**: Check scan status until completion
4. **Validate Quality**: Verify quality gate compliance

### Multi-Branch Analysis

```yaml
# Branch-specific project keys
projectKey: "my-app:${BRANCH_NAME}"
# Example: "my-app:main", "my-app:feature/auth"
```

### Pull Request Analysis

```yaml
# PR-specific analysis
projectKey: "my-app:PR-${PR_NUMBER}"
# Additional PR-specific parameters would be configured
```

### Continuous Integration

```yaml
# Daily/nightly scans with timestamps
projectKey: "my-app:nightly-${DATE}"
# Automated cleanup of old scan results
```

## Quality Gate Configuration

### Standard Quality Gates

SonarQube quality gates typically include:

- **Coverage**: Minimum code coverage percentage
- **Duplicated Lines**: Maximum allowed code duplication
- **Maintainability Rating**: Code maintainability score
- **Reliability Rating**: Bug-free code rating
- **Security Rating**: Security vulnerability assessment

### Custom Quality Gates

```yaml
# Example quality gate criteria
- Coverage > 80%
- Duplicated Lines < 5%
- Critical Issues = 0
- Major Issues < 10
- Security Hotspots = 0
```

## Security Considerations

### Token Management

- Use secure token storage (secrets management)
- Rotate authentication tokens regularly
- Apply principle of least privilege for token permissions
- Never expose tokens in logs or console output

### Network Security

- Use HTTPS for SonarQube communication
- Configure proper firewall rules
- Implement VPN/network isolation where required
- Validate SSL certificates in production

### Access Control

- Configure project-level permissions in SonarQube
- Use service accounts for automated scans
- Implement audit logging for scan activities
- Regular review of user permissions and access

## Best Practices

### Project Organization

- Use consistent project key naming conventions
- Group related projects logically
- Document project purposes and ownership
- Regular cleanup of unused projects

### Scan Management

- Schedule scans during off-peak hours for large codebases
- Configure appropriate scan timeouts
- Monitor scan performance and resource usage
- Implement scan result notifications

### Quality Management

- Define clear quality standards and gates
- Regular review and update of quality gate criteria
- Establish remediation processes for quality gate failures
- Track quality trends over time

### CI/CD Integration

- Fail builds on quality gate failures
- Provide clear feedback on quality issues
- Integrate with issue tracking systems
- Automate quality reporting and metrics

## Troubleshooting

### Common Issues

- **Authentication failures**: Verify token validity and permissions
- **Network connectivity**: Check firewall and DNS resolution
- **Scan timeouts**: Adjust timeout settings for large codebases
- **Quality gate failures**: Review and address code quality issues

### Error Handling

- Implement retry mechanisms for transient failures
- Log detailed error information for debugging
- Provide meaningful error messages to developers
- Establish escalation procedures for persistent issues

## Notes

- All activities in this package use console handlers for output and validation
- These activities provide structure and validation for SonarQube operations
- For production use, consider implementing corresponding PipelineRef handlers with actual SonarQube API integration
- Ensure proper SonarQube server access and authentication before using these activities
- Consider implementing webhook notifications for scan completion events
- Regular monitoring of SonarQube server performance and capacity is recommended
