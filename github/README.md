# GitHub

## Overview

Building blocks for integrating with GitHub repositories, organizations, and development workflows. This package provides comprehensive activities for GitHub operations including repository management, project scaffolding from templates, and repository listing capabilities using GitHub's API and command-line tools.

## Components

### Activities

#### Repository Management

- **List Repositories** (`github.activity.tekton.listRepositories`): List repositories from a GitHub organization using GitHub CLI
- **Create Spring Boot from Template** (`github.activity.tekton.springBootScaffolder`): Create a new Spring Boot project repository from a GitHub template
- **Create Spring Boot from Initializr** (`github.activity.tekton.createSpringBootFromInitializr`): Create a Spring Boot project using Spring Initializr and push to GitHub

### Models

- **GitHub Organization** (`github.organisation`): Represents a GitHub organization with metadata and settings
- **GitHub Repository** (`github.repository`): Represents a GitHub repository with configuration and properties
- **GitHub Credentials** (`github.credentials`): Personal Access Token credentials for GitHub API access

### PipelineRefs

- **GitHub Script Pipeline** (`github.pipelineRef.gh-script`): Tekton pipeline for executing GitHub CLI commands

## Dependencies

### Required

None - this package has no required dependencies.

### Optional

None - this package currently has no optional dependencies.

## Usage

### Repository Listing

```yaml
# List repositories from organization
activity: github.activity.tekton.listRepositories
inputs:
  githubToken:
    type: github.secret.personalAccessToken
    value: "ghp_xxxxxxxxxxxxxxxxxxxx"
  organization: "my-organization"
```

### Project Scaffolding

```yaml
# Create Spring Boot project from GitHub template
activity: github.activity.tekton.springBootScaffolder
inputs:
  githubToken:
    type: github.secret.personalAccessToken
    value: "ghp_xxxxxxxxxxxxxxxxxxxx"
  organization: "my-organization"
  projectName: "my-new-service"
  templateProject: "hmcts/spring-boot-template"

# Create Spring Boot project from Spring Initializr
activity: github.activity.tekton.createSpringBootFromInitializr
inputs:
  githubToken:
    type: github.secret.personalAccessToken
    value: "ghp_xxxxxxxxxxxxxxxxxxxx"
  organization: "my-organization"
  projectName: "my-spring-app"
  groupId: "com.mycompany"
  artifactId: "my-spring-app"
  version: "1.0.0"
  javaVersion: "17"
  dependencies: "web,jpa,h2"
```

### Complete Development Workflow

```yaml
# End-to-end repository setup and scaffolding
steps:
  - name: List Existing Repositories
    activity: github.activity.tekton.listRepositories
    inputs:
      githubToken: "${GITHUB_TOKEN}"
      organization: "${GITHUB_ORG}"

  - name: Create New Service Repository
    activity: github.activity.tekton.springBootScaffolder
    inputs:
      githubToken: "${GITHUB_TOKEN}"
      organization: "${GITHUB_ORG}"
      projectName: "payment-service"
      templateProject: "hmcts/spring-boot-template"

  - name: Create Frontend Repository
    activity: github.activity.tekton.createSpringBootFromInitializr
    inputs:
      githubToken: "${GITHUB_TOKEN}"
      organization: "${GITHUB_ORG}"
      projectName: "payment-ui"
      groupId: "com.company.payment"
      artifactId: "payment-ui"
      dependencies: "web,thymeleaf,security"
```

## Configuration

### Required Configuration

All activities require GitHub API access:

- **githubToken**: Personal Access Token with appropriate repository permissions
- **organization**: Target GitHub organization name

### Authentication

These activities use Tekton pipelines with GitHub CLI integration. You need:

- GitHub Personal Access Token with repository permissions:
  - `repo` scope for private repositories
  - `public_repo` scope for public repositories
  - `read:org` scope for organization access
- GitHub CLI (`gh`) installed in the Tekton environment
- Network connectivity to GitHub API

### Activity-Specific Configuration

#### Repository Listing

- **organization**: GitHub organization name (defaults to "clrslate")
- **githubToken**: Must have read access to organization repositories

#### Project Scaffolding from Templates

- **templateProject**: Source template repository (e.g., "hmcts/spring-boot-template")
- **projectName**: New repository name (must be unique in organization)
- **organization**: Target GitHub organization for new repository

#### Spring Initializr Integration

- **groupId**: Maven group ID for the project (e.g., "com.company.app")
- **artifactId**: Maven artifact ID (usually same as project name)
- **version**: Initial project version (defaults to "0.0.1-SNAPSHOT")
- **javaVersion**: Java version (e.g., "11", "17", "21")
- **dependencies**: Comma-separated list of Spring Boot starters

## GitHub Integration Patterns

### Project Templates

Use GitHub templates for consistent project structure:

1. **Create Template Repository**: Set up base project structure
2. **Enable Template**: Mark repository as template in GitHub settings
3. **Use Template**: Reference template in scaffolding activities
4. **Customize**: Configure project-specific settings after creation

### Organization Management

```yaml
# Multi-team organization structure
teams:
  - name: backend-team
    repositories: ["api-gateway", "user-service", "payment-service"]

  - name: frontend-team
    repositories: ["web-app", "mobile-app", "admin-portal"]

  - name: platform-team
    repositories: ["infrastructure", "monitoring", "ci-cd-tools"]
```

### Development Workflows

```yaml
# Automated project initialization
project_setup:
  - name: Create Repository
    activity: github.activity.tekton.springBootScaffolder

  - name: Configure Branch Protection
    # Additional configuration activities would go here

  - name: Setup CI/CD
    # GitHub Actions or other CI/CD setup
```

## Best Practices

### Security Considerations

- Store GitHub tokens securely using secrets management
- Use fine-grained Personal Access Tokens with minimal required permissions
- Rotate tokens regularly and audit access
- Never expose tokens in logs or console output

### Repository Management

- Use consistent naming conventions for repositories
- Implement proper branch protection rules
- Configure code review requirements
- Use repository templates for standardization

### Project Scaffolding

- Maintain up-to-date project templates
- Include comprehensive README files in templates
- Configure default branch protection and CI/CD
- Document template usage and customization

### Organization Standards

- Establish coding standards and guidelines
- Implement consistent project structure
- Use standardized dependency management
- Maintain documentation and examples

## Integration with CI/CD

### GitHub Actions Integration

```yaml
# Trigger GitHub Actions workflows after repository creation
post_creation:
  - name: Enable GitHub Actions
  - name: Configure Workflow Templates
  - name: Set Repository Secrets
  - name: Configure Environment Protection
```

### Multi-Repository Operations

```yaml
# Batch operations across multiple repositories
batch_operations:
  - name: List All Repositories
    activity: github.activity.tekton.listRepositories

  - name: Process Each Repository
    # Iterate through repository list for bulk operations
```

## Troubleshooting

### Common Issues

- **Authentication failures**: Verify token permissions and expiration
- **Repository creation errors**: Check organization permissions and name conflicts
- **Template access issues**: Verify template repository visibility and permissions
- **Network connectivity**: Ensure GitHub API access from Tekton environment

### Error Handling

- Implement retry mechanisms for transient API failures
- Log detailed error information for debugging
- Provide meaningful error messages for common scenarios
- Establish escalation procedures for persistent issues

### Token Management

- Use service accounts for automated operations
- Implement token rotation policies
- Monitor token usage and rate limits
- Configure proper secret storage and access

## Notes

- All activities use Tekton pipelines for execution
- GitHub CLI (`gh`) is required in the execution environment
- Activities provide structure and validation for GitHub operations
- Ensure proper GitHub organization access and permissions before using activities
- Consider implementing webhook notifications for repository events
- Regular review of organization repositories and access is recommended
