# Azure DevOps

## Overview

Building blocks for managing Azure DevOps projects, pipelines, repositories, and artifacts. This package provides a comprehensive set of activities for common Azure DevOps operations including project management, CI/CD pipeline operations, repository management, artifact publishing, and work item tracking.

## Components

### Activities

- **Create Project** (`azureDevOps.project.createProject`): Create a new Azure DevOps project with customizable visibility settings
- **Run Pipeline** (`azureDevOps.pipeline.runPipeline`): Trigger pipeline runs with custom parameters and branch selection
- **Create Repository** (`azureDevOps.repository.createRepository`): Create Git repositories within Azure DevOps projects
- **Publish Artifact** (`azureDevOps.artifact.publishArtifact`): Publish build artifacts to Azure DevOps artifact feeds
- **Create Pipeline** (`azureDevOps.pipeline.createPipeline`): Create new CI/CD pipelines linked to repositories
- **Create Work Item** (`azureDevOps.board.createWorkItem`): Create work items in Azure Boards for project tracking

## Dependencies

### Required

None - this package has no required dependencies.

### Optional

None - this package currently has no optional dependencies.

## Usage

### Basic Project Setup

```yaml
# Create a new Azure DevOps project
activity: azureDevOps.project.createProject
inputs:
  organizationName: "my-organization"
  projectName: "my-new-project"
  description: "A sample project for demonstration"
  visibility: "private"
```

### Repository Management

```yaml
# Create a new repository
activity: azureDevOps.repository.createRepository
inputs:
  organizationName: "my-organization"
  projectName: "my-project"
  repoName: "my-app-repo"
  initializeWithReadme: "true"
```

### Pipeline Operations

```yaml
# Create a new pipeline
activity: azureDevOps.pipeline.createPipeline
inputs:
  organizationName: "my-organization"
  projectName: "my-project"
  pipelineName: "Build Pipeline"
  repository: "https://github.com/myorg/myrepo"
  yamlPath: "build/azure-pipelines.yml"

# Run an existing pipeline
activity: azureDevOps.pipeline.runPipeline
inputs:
  organizationName: "my-organization"
  projectName: "my-project"
  pipelineId: "123"
  branch: "develop"
  parameters: '{"buildConfiguration": "Release"}'
```

### Work Item Management

```yaml
# Create a work item
activity: azureDevOps.board.createWorkItem
inputs:
  organizationName: "my-organization"
  projectName: "my-project"
  workItemType: "User Story"
  title: "Implement new feature"
  description: "Add authentication to the application"
  assignedTo: "developer@company.com"
```

### Artifact Publishing

```yaml
# Publish build artifacts
activity: azureDevOps.artifact.publishArtifact
inputs:
  organizationName: "my-organization"
  projectName: "my-project"
  artifactName: "build-output"
  artifactPath: "./dist"
  pipelineId: "123"
```

## Configuration

### Required Configuration

All activities require the following base configuration:

- **organizationName**: Your Azure DevOps organization name
- **projectName**: Target Azure DevOps project name

### Authentication

These activities use console handlers for demonstration and validation purposes. In a production environment, you would need:

- Azure DevOps Personal Access Token (PAT)
- Appropriate permissions for the target organization and projects
- Service connection configuration for automated operations

### Activity-Specific Configuration

#### Project Creation

- **visibility**: Set to "public" or "private" based on project requirements
- **description**: Provide meaningful project descriptions for team clarity

#### Pipeline Operations

- **branch**: Specify target branches for pipeline runs (defaults to "main")
- **parameters**: Use JSON format for custom pipeline parameters
- **yamlPath**: Ensure YAML pipeline definitions exist at specified paths

#### Repository Management

- **initializeWithReadme**: Set to "true" for repositories that need initial content

#### Work Item Management

- **workItemType**: Choose appropriate work item types based on your process template
- **assignedTo**: Use valid user email addresses or display names

## Notes

- All activities in this package use console handlers for output and validation
- These activities provide structure and validation for Azure DevOps operations
- For production use, consider implementing corresponding PipelineRef handlers with actual Azure DevOps API integration
- Ensure proper access permissions are configured in your Azure DevOps organization before using these activities
