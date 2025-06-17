# Jenkins

## Overview

Consume Jenkins API. This package provides a comprehensive set of activities for interacting with Jenkins automation server, including job management, build operations, and instance administration through the Jenkins API.

## Components

### Activities

#### Job Management

- **Copy Job** (`jenkins.job.copyJob`): Copy a specific job in Jenkins
- **Create Job** (`jenkins.job.createJob`): Create a new job in Jenkins
- **Trigger Job** (`jenkins.job.triggerJob`): Trigger a specific job in Jenkins
- **Trigger Job with Parameters** (`jenkins.job.triggerJobWithParams`): Trigger a specific job with parameters

#### Instance Management

- **Quiet Down Instance** (`jenkins.instance.quietDownInstance`): Put Jenkins in quiet mode
- **Restart Instance** (`jenkins.instance.restartInstance`): Restart Jenkins immediately
- **Safely Restart Instance** (`jenkins.instance.safeRestartInstance`): Restart Jenkins once no jobs are running
- **Shutdown Instance** (`jenkins.instance.shutdownInstance`): Shutdown Jenkins immediately

#### Build Operations

- **Get Many Builds** (`jenkins.build.getAllBuilds`): List all builds for a job

## Dependencies

### Required

None - this package has no required dependencies.

### Optional

None - this package currently has no optional dependencies.

## Usage

### Job Management

```yaml
# Create a new job
activity: jenkins.job.createJob
inputs:
  newJob: "my-new-pipeline"
  xml: |
    <?xml version='1.1' encoding='UTF-8'?>
    <project>
      <description>My new pipeline job</description>
      <keepDependencies>false</keepDependencies>
      <properties/>
      <scm class="hudson.scm.NullSCM"/>
      <canRoam>true</canRoam>
      <disabled>false</disabled>
      <blockBuildWhenDownstreamBuilding>false</blockBuildWhenDownstreamBuilding>
      <blockBuildWhenUpstreamBuilding>false</blockBuildWhenUpstreamBuilding>
      <triggers/>
      <concurrentBuild>false</concurrentBuild>
      <builders/>
      <publishers/>
      <buildWrappers/>
    </project>

# Copy an existing job
activity: jenkins.job.copyJob
inputs:
  job: "existing-pipeline"
  newJob: "copied-pipeline"

# Trigger a job
activity: jenkins.job.triggerJob
inputs:
  job: "my-pipeline"

# Trigger a job with parameters
activity: jenkins.job.triggerJobWithParams
inputs:
  job: "parameterized-pipeline"
  params: '[{"name": "BRANCH", "value": "main"}, {"name": "ENVIRONMENT", "value": "staging"}]'
```

### Build Operations

```yaml
# Get all builds for a job
activity: jenkins.build.getAllBuilds
inputs:
  job: "my-pipeline"
  returnAll: "false"
  limit: "25"

# Get all builds without limit
activity: jenkins.build.getAllBuilds
inputs:
  job: "critical-pipeline"
  returnAll: "true"
  limit: "100"
```

### Instance Management

```yaml
# Put Jenkins in quiet mode for maintenance
activity: jenkins.instance.quietDownInstance
inputs:
  reason: "Scheduled maintenance window"

# Safely restart Jenkins (waits for jobs to complete)
activity: jenkins.instance.safeRestartInstance

# Immediate restart (use with caution)
activity: jenkins.instance.restartInstance

# Shutdown Jenkins
activity: jenkins.instance.shutdownInstance
```

### CI/CD Pipeline Integration

```yaml
# Complete pipeline deployment workflow
steps:
  - name: Create Pipeline Job
    activity: jenkins.job.createJob
    inputs:
      newJob: "deployment-pipeline-${ENV}"
      xml: "${PIPELINE_XML_CONFIG}"

  - name: Trigger Initial Build
    activity: jenkins.job.triggerJobWithParams
    inputs:
      job: "deployment-pipeline-${ENV}"
      params: '[{"name": "VERSION", "value": "${BUILD_VERSION}"}]'

  - name: Monitor Build Progress
    activity: jenkins.build.getAllBuilds
    inputs:
      job: "deployment-pipeline-${ENV}"
      returnAll: "false"
      limit: "1"
```

## Configuration

### Required Configuration

Activities require Jenkins API connectivity:

- **Jenkins server URL**: Base URL of your Jenkins instance
- **Authentication**: Valid Jenkins credentials or API tokens
- **Network access**: Connectivity to Jenkins server

### Authentication

These activities use console handlers for demonstration and validation purposes. In a production environment, you would need:

- Jenkins API authentication (username/password or API token)
- Proper user permissions for job and instance management
- Network connectivity to Jenkins server

### Activity-Specific Configuration

#### Job Management

- **XML Configuration**: Valid Jenkins job configuration XML for job creation
- **Job Names**: Must follow Jenkins naming conventions (no special characters)
- **Parameters**: Use JSON array format for parameterized job triggers

#### Instance Management

- **Administrative Privileges**: Instance management requires Jenkins admin permissions
- **Maintenance Planning**: Use quiet mode before restarts or shutdowns
- **Safety Considerations**: Always prefer safe restart over immediate restart

#### Build Operations

- **Result Limits**: Configure appropriate limits for build history retrieval
- **Job Names**: Ensure job names exist and are accessible

## Jenkins Integration Patterns

### CI/CD Automation

1. **Job Setup**: Create and configure pipeline jobs
2. **Trigger Builds**: Execute jobs with appropriate parameters
3. **Monitor Progress**: Track build status and history
4. **Maintenance**: Manage instance lifecycle safely

### Multi-Environment Deployment

```yaml
# Environment-specific job management
environments:
  - name: development
    job: "app-deploy-dev"
    params: '[{"name": "ENV", "value": "dev"}]'

  - name: staging
    job: "app-deploy-staging"
    params: '[{"name": "ENV", "value": "staging"}]'

  - name: production
    job: "app-deploy-prod"
    params: '[{"name": "ENV", "value": "prod"}]'
```

### Automated Maintenance

```yaml
# Scheduled maintenance workflow
steps:
  - name: Enter Quiet Mode
    activity: jenkins.instance.quietDownInstance
    inputs:
      reason: "Automated maintenance - plugin updates"

  - name: Wait for Jobs to Complete
    # Custom wait logic would go here

  - name: Safe Restart
    activity: jenkins.instance.safeRestartInstance
```

## Best Practices

### Job Management

- Use descriptive job names with consistent naming conventions
- Store job configurations in version control
- Implement proper job organization with folders and views
- Regular backup of Jenkins job configurations

### Build Operations

- Monitor build queues and execution times
- Implement build artifact cleanup policies
- Use appropriate build retention policies
- Configure build notifications and alerts

### Instance Management

- Always use quiet mode before maintenance
- Prefer safe restart over immediate restart
- Schedule maintenance during low-activity periods
- Maintain proper backup and disaster recovery procedures

### Security Considerations

- Use API tokens instead of passwords
- Implement proper user access controls
- Secure Jenkins with HTTPS
- Regular security updates and plugin management

## CI/CD Workflows

### Build Pipeline Management

```yaml
# Multi-stage pipeline setup
stages:
  - name: Setup Pipeline
    activity: jenkins.job.createJob
    inputs:
      newJob: "multi-stage-pipeline"
      xml: "${MULTI_STAGE_XML}"

  - name: Trigger Build
    activity: jenkins.job.triggerJobWithParams
    inputs:
      job: "multi-stage-pipeline"
      params: '[{"name": "STAGE", "value": "build"}]'

  - name: Trigger Test
    activity: jenkins.job.triggerJobWithParams
    inputs:
      job: "multi-stage-pipeline"
      params: '[{"name": "STAGE", "value": "test"}]'

  - name: Trigger Deploy
    activity: jenkins.job.triggerJobWithParams
    inputs:
      job: "multi-stage-pipeline"
      params: '[{"name": "STAGE", "value": "deploy"}]'
```

### Automated Job Provisioning

```yaml
# Create standardized job templates
job_templates:
  - name: microservice
    activity: jenkins.job.createJob
    template: microservice-pipeline.xml

  - name: frontend
    activity: jenkins.job.createJob
    template: frontend-pipeline.xml

  - name: integration-test
    activity: jenkins.job.createJob
    template: integration-test.xml
```

## Troubleshooting

### Common Issues

- **Authentication failures**: Verify API credentials and permissions
- **Job creation errors**: Validate XML configuration syntax
- **Build trigger failures**: Check job existence and parameter formats
- **Instance management errors**: Ensure administrative privileges

### Error Handling

- Implement retry mechanisms for transient failures
- Log detailed error information for debugging
- Provide meaningful error messages for operations teams
- Establish escalation procedures for critical failures

### Monitoring and Maintenance

- Regular health checks of Jenkins instance
- Monitor job execution times and success rates
- Automated alerting for failed builds or instance issues
- Periodic review of job configurations and cleanup

## XML Configuration Examples

### Basic Pipeline Job

```xml
<?xml version='1.1' encoding='UTF-8'?>
<flow-definition plugin="workflow-job">
  <description>Basic pipeline job</description>
  <keepDependencies>false</keepDependencies>
  <properties/>
  <definition class="org.jenkinsci.plugins.workflow.cps.CpsFlowDefinition" plugin="workflow-cps">
    <script>
      pipeline {
        agent any
        stages {
          stage('Build') {
            steps {
              echo 'Building...'
            }
          }
          stage('Test') {
            steps {
              echo 'Testing...'
            }
          }
          stage('Deploy') {
            steps {
              echo 'Deploying...'
            }
          }
        }
      }
    </script>
    <sandbox>true</sandbox>
  </definition>
  <triggers/>
  <disabled>false</disabled>
</flow-definition>
```

### Parameterized Job

```xml
<?xml version='1.1' encoding='UTF-8'?>
<project>
  <description>Parameterized build job</description>
  <keepDependencies>false</keepDependencies>
  <properties>
    <hudson.model.ParametersDefinitionProperty>
      <parameterDefinitions>
        <hudson.model.StringParameterDefinition>
          <name>BRANCH</name>
          <description>Git branch to build</description>
          <defaultValue>main</defaultValue>
        </hudson.model.StringParameterDefinition>
        <hudson.model.ChoiceParameterDefinition>
          <name>ENVIRONMENT</name>
          <description>Target environment</description>
          <choices>
            <string>dev</string>
            <string>staging</string>
            <string>prod</string>
          </choices>
        </hudson.model.ChoiceParameterDefinition>
      </parameterDefinitions>
    </hudson.model.ParametersDefinitionProperty>
  </properties>
  <builders>
    <hudson.tasks.Shell>
      <command>echo "Building branch $BRANCH for $ENVIRONMENT"</command>
    </hudson.tasks.Shell>
  </builders>
</project>
```

## Notes

- All activities in this package use console handlers for output and validation
- These activities provide structure and validation for Jenkins API operations
- For production use, consider implementing corresponding PipelineRef handlers with actual Jenkins API integration
- Ensure proper Jenkins server access and authentication before using these activities
- Consider implementing webhook notifications for build status updates
- Regular monitoring of Jenkins server performance and capacity is recommended
