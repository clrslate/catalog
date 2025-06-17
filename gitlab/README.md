# GitLab

## Overview

Building blocks for managing GitLab repositories, pipelines, and projects. This package provides a comprehensive set of activities for GitLab DevOps workflows, including repository management, CI/CD pipeline execution, issue tracking, merge request handling, and organizational group management.

## Components

### Activities

#### Repository Management

- **Create Repository** (`gitlab.repo.createRepository`): Create a new repository in a GitLab project

#### Pipeline Management

- **Run Pipeline** (`gitlab.pipeline.runPipeline`): Trigger a pipeline run in GitLab

#### Issue Management

- **Create Issue** (`gitlab.issue.createIssue`): Create a new issue in a GitLab repository

#### Merge Request Management

- **Create Merge Request** (`gitlab.mergeRequest.createMergeRequest`): Create a merge request in a GitLab repository

#### Group Management

- **Create Group** (`gitlab.group.createGroup`): Create a new group in GitLab

## Dependencies

### Required

None - this package has no required dependencies.

### Optional

None - this package currently has no optional dependencies.

## Usage

### Repository Management

```yaml
# Create a new repository
activity: gitlab.repo.createRepository
inputs:
  repoName: "my-awesome-project"
  visibility: "private"

# Create a public repository
activity: gitlab.repo.createRepository
inputs:
  repoName: "open-source-library"
  visibility: "public"
```

### Pipeline Management

```yaml
# Trigger pipeline on main branch
activity: gitlab.pipeline.runPipeline
inputs:
  projectId: "12345"
  branch: "main"

# Trigger pipeline on feature branch
activity: gitlab.pipeline.runPipeline
inputs:
  projectId: "12345"
  branch: "feature/new-functionality"

# Trigger pipeline on development branch
activity: gitlab.pipeline.runPipeline
inputs:
  projectId: "67890"
  branch: "develop"
```

### Issue Management

```yaml
# Create a bug report
activity: gitlab.issue.createIssue
inputs:
  projectId: "12345"
  title: "Fix login validation error"

# Create a feature request
activity: gitlab.issue.createIssue
inputs:
  projectId: "12345"
  title: "Add dark mode support"

# Create a technical debt issue
activity: gitlab.issue.createIssue
inputs:
  projectId: "12345"
  title: "Refactor authentication module"
```

### Merge Request Management

```yaml
# Create a merge request for feature
activity: gitlab.mergeRequest.createMergeRequest
inputs:
  projectId: "12345"
  sourceBranch: "feature/user-authentication"
  targetBranch: "main"
  title: "Add user authentication system"

# Create a merge request for bugfix
activity: gitlab.mergeRequest.createMergeRequest
inputs:
  projectId: "12345"
  sourceBranch: "bugfix/login-issue"
  targetBranch: "develop"
  title: "Fix login validation bug"

# Create a merge request with custom target
activity: gitlab.mergeRequest.createMergeRequest
inputs:
  projectId: "12345"
  sourceBranch: "hotfix/critical-security-fix"
  targetBranch: "release/v2.1"
  title: "Critical security vulnerability fix"
```

### Group Management

```yaml
# Create a private team group
activity: gitlab.group.createGroup
inputs:
  groupName: "backend-team"
  visibility: "private"

# Create a public organization group
activity: gitlab.group.createGroup
inputs:
  groupName: "open-source-projects"
  visibility: "public"

# Create a department group
activity: gitlab.group.createGroup
inputs:
  groupName: "engineering-department"
  visibility: "private"
```

### Complete DevOps Workflow

```yaml
# Complete GitLab project setup and development workflow
steps:
  - name: Create Development Group
    activity: gitlab.group.createGroup
    inputs:
      groupName: "product-development"
      visibility: "private"

  - name: Create Project Repository
    activity: gitlab.repo.createRepository
    inputs:
      repoName: "web-application"
      visibility: "private"

  - name: Create Initial Issue
    activity: gitlab.issue.createIssue
    inputs:
      projectId: "${PROJECT_ID}"
      title: "Set up initial project structure"

  - name: Trigger Initial Pipeline
    activity: gitlab.pipeline.runPipeline
    inputs:
      projectId: "${PROJECT_ID}"
      branch: "main"

  - name: Create Feature Branch MR
    activity: gitlab.mergeRequest.createMergeRequest
    inputs:
      projectId: "${PROJECT_ID}"
      sourceBranch: "feature/initial-setup"
      targetBranch: "main"
      title: "Initial project setup and configuration"
```

## Configuration

### Required Configuration

Activities require GitLab API connectivity:

- **projectId**: GitLab project identifier
- **groupName**: Name for new groups
- **repoName**: Repository name for new repositories

### Authentication

These activities use console handlers for demonstration and validation purposes. In a production environment, you would need:

- GitLab API authentication token with appropriate permissions
- Network connectivity to GitLab instance
- Proper user permissions for repository, pipeline, issue, and group management

### Activity-Specific Configuration

#### Repository Management

- **repoName**: Must follow GitLab naming conventions
- **visibility**: Choose between "public" and "private" access

#### Pipeline Management

- **projectId**: Must be a valid GitLab project identifier
- **branch**: Target branch for pipeline execution

#### Issue Management

- **projectId**: Target project for issue creation
- **title**: Descriptive issue title

#### Merge Request Management

- **projectId**: Target project for merge request
- **sourceBranch**: Feature or development branch
- **targetBranch**: Destination branch (usually main/develop)
- **title**: Descriptive merge request title

#### Group Management

- **groupName**: Must follow GitLab group naming conventions
- **visibility**: Choose between "public" and "private" access

## GitLab DevOps Patterns

### GitFlow Workflow

```yaml
# GitFlow development pattern
gitflow_workflow:
  branches:
    main: "production-ready code"
    develop: "integration branch"
    feature: "feature/feature-name"
    release: "release/version-number"
    hotfix: "hotfix/issue-description"

  merge_requests:
    - source: "feature/*"
      target: "develop"
    - source: "develop"
      target: "main"
    - source: "hotfix/*"
      target: "main"
```

### CI/CD Pipeline Integration

```yaml
# Automated CI/CD workflow
cicd_integration:
  - name: Code Commit
    trigger: "push to feature branch"

  - name: Create Merge Request
    activity: gitlab.mergeRequest.createMergeRequest

  - name: Run Pipeline
    activity: gitlab.pipeline.runPipeline

  - name: Deploy on Success
    condition: "pipeline success"
```

### Issue-Driven Development

```yaml
# Issue-driven development workflow
issue_workflow:
  - name: Create Feature Issue
    activity: gitlab.issue.createIssue

  - name: Create Feature Branch
    branch: "feature/issue-${ISSUE_NUMBER}"

  - name: Development Work
    process: "implement feature"

  - name: Create Merge Request
    activity: gitlab.mergeRequest.createMergeRequest
    reference: "Closes #${ISSUE_NUMBER}"
```

## Best Practices

### Repository Management

- Use descriptive repository names that reflect project purpose
- Set appropriate visibility based on project requirements
- Implement consistent naming conventions across projects
- Configure proper branch protection rules

### Pipeline Management

- Design pipelines with clear stages (build, test, deploy)
- Use pipeline variables for environment-specific configurations
- Implement proper error handling and notifications
- Cache dependencies to improve pipeline performance

### Issue Management

- Use clear, descriptive issue titles
- Apply appropriate labels and milestones
- Link issues to merge requests for traceability
- Regular issue triage and prioritization

### Merge Request Management

- Write clear, descriptive merge request titles
- Include detailed descriptions of changes
- Use merge request templates for consistency
- Require code reviews before merging

### Group Management

- Organize groups by teams, departments, or projects
- Set appropriate permissions and access levels
- Use subgroups for hierarchical organization
- Regular access review and cleanup

## Development Workflows

### Feature Development

```yaml
# Standard feature development workflow
feature_workflow:
  - name: Create Feature Issue
    activity: gitlab.issue.createIssue
    inputs:
      title: "Implement user dashboard"

  - name: Create Feature Branch
    branch: "feature/user-dashboard"

  - name: Trigger Development Pipeline
    activity: gitlab.pipeline.runPipeline
    inputs:
      branch: "feature/user-dashboard"

  - name: Create Merge Request
    activity: gitlab.mergeRequest.createMergeRequest
    inputs:
      sourceBranch: "feature/user-dashboard"
      targetBranch: "develop"
      title: "Add user dashboard functionality"
```

### Release Management

```yaml
# Release preparation workflow
release_workflow:
  - name: Create Release Branch
    branch: "release/v2.0.0"

  - name: Run Release Pipeline
    activity: gitlab.pipeline.runPipeline
    inputs:
      branch: "release/v2.0.0"

  - name: Create Release MR to Main
    activity: gitlab.mergeRequest.createMergeRequest
    inputs:
      sourceBranch: "release/v2.0.0"
      targetBranch: "main"
      title: "Release v2.0.0"

  - name: Create Backmerge MR to Develop
    activity: gitlab.mergeRequest.createMergeRequest
    inputs:
      sourceBranch: "main"
      targetBranch: "develop"
      title: "Backmerge release v2.0.0 to develop"
```

### Hotfix Process

```yaml
# Emergency hotfix workflow
hotfix_workflow:
  - name: Create Hotfix Issue
    activity: gitlab.issue.createIssue
    inputs:
      title: "Critical security vulnerability fix"

  - name: Create Hotfix Branch
    branch: "hotfix/security-vulnerability"

  - name: Run Hotfix Pipeline
    activity: gitlab.pipeline.runPipeline
    inputs:
      branch: "hotfix/security-vulnerability"

  - name: Create Hotfix MR
    activity: gitlab.mergeRequest.createMergeRequest
    inputs:
      sourceBranch: "hotfix/security-vulnerability"
      targetBranch: "main"
      title: "Critical security vulnerability fix"
```

## Team Organization

### Multi-Team Structure

```yaml
# Organizational structure
organization:
  groups:
    - name: "engineering"
      visibility: "private"
      subgroups:
        - "frontend-team"
        - "backend-team"
        - "devops-team"

    - name: "product"
      visibility: "private"
      subgroups:
        - "design-team"
        - "product-management"
```

### Project Organization

```yaml
# Project categorization
projects:
  web_applications:
    - "customer-portal"
    - "admin-dashboard"
    - "public-website"

  mobile_applications:
    - "ios-app"
    - "android-app"

  infrastructure:
    - "deployment-scripts"
    - "monitoring-tools"
```

## Integration Patterns

### Automated Project Setup

```yaml
# Automated new project initialization
project_initialization:
  - name: Create Project Group
    activity: gitlab.group.createGroup
    inputs:
      groupName: "${PROJECT_NAME}-team"

  - name: Create Main Repository
    activity: gitlab.repo.createRepository
    inputs:
      repoName: "${PROJECT_NAME}"

  - name: Create Documentation Repository
    activity: gitlab.repo.createRepository
    inputs:
      repoName: "${PROJECT_NAME}-docs"

  - name: Create Initial Issues
    activity: gitlab.issue.createIssue
    inputs:
      title: "Project setup and initial configuration"
```

### Continuous Integration Setup

```yaml
# CI/CD pipeline automation
ci_setup:
  - name: Trigger Main Pipeline
    activity: gitlab.pipeline.runPipeline
    inputs:
      branch: "main"

  - name: Trigger Feature Pipelines
    activity: gitlab.pipeline.runPipeline
    inputs:
      branch: "feature/*"

  - name: Create Pipeline Status Issue
    activity: gitlab.issue.createIssue
    condition: "pipeline failure"
    inputs:
      title: "Pipeline failure on ${BRANCH}"
```

## Troubleshooting

### Common Issues

- **Authentication failures**: Verify GitLab API token and permissions
- **Project access errors**: Check user permissions for target projects
- **Pipeline trigger failures**: Validate project ID and branch existence
- **Merge request conflicts**: Ensure source and target branches exist

### Error Handling

- Implement retry mechanisms for transient API failures
- Log detailed error information for debugging
- Provide meaningful error messages for development teams
- Establish escalation procedures for critical pipeline failures

### Performance Optimization

- Use GitLab API efficiently with appropriate rate limiting
- Batch operations when possible to reduce API calls
- Implement caching for frequently accessed project information
- Monitor pipeline execution times and optimize bottlenecks

## Security Best Practices

### Access Control

- Use least privilege principle for API tokens
- Implement proper group and project access controls
- Regular audit of user permissions and access
- Use protected branches for critical code paths

### Code Security

- Implement mandatory code reviews through merge requests
- Use GitLab security scanning features
- Configure branch protection rules
- Implement signed commits for critical repositories

### Pipeline Security

- Secure pipeline variables and secrets
- Use GitLab's built-in secret detection
- Implement proper artifact handling
- Regular security scanning in pipelines

## Multi-Environment Management

### Development Environments

```yaml
development:
  gitlab_instance: "https://gitlab-dev.company.com"
  default_visibility: "private"
  pipeline_triggers: "all_branches"
```

### Staging Environments

```yaml
staging:
  gitlab_instance: "https://gitlab-staging.company.com"
  default_visibility: "private"
  pipeline_triggers: "develop_and_release_branches"
```

### Production Environments

```yaml
production:
  gitlab_instance: "https://gitlab.company.com"
  default_visibility: "private"
  pipeline_triggers: "main_branch_only"
  additional_security:
    - "mandatory_code_review"
    - "signed_commits"
    - "protected_branches"
```

## Monitoring and Analytics

### Key Metrics

- Repository creation and usage patterns
- Pipeline success/failure rates
- Merge request turnaround times
- Issue resolution times
- Code review participation

### Automated Reporting

- Daily pipeline status reports
- Weekly merge request summaries
- Monthly repository activity reports
- Quarterly security audit reports

## Notes

- All activities in this package use console handlers for output and validation
- These activities provide structure and validation for GitLab operations
- For production use, consider implementing corresponding PipelineRef handlers with actual GitLab API integration
- Ensure proper GitLab instance access and authentication before using these activities
- Consider implementing webhook integrations for real-time event handling
- Regular backup and disaster recovery planning for GitLab data is recommended
