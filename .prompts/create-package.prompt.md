---
mode: 'agent'
description: 'Create a new ClrSlate package with standardized structure and metadata'
---

# ClrSlate Package Creation Instructions

Your goal is to create a new ClrSlate package following the standardized structure and metadata requirements.

## Information Gathering

First, collect the following required information from the user:

### Required Package Information
1. **Package Name**: Must use kebab-case (e.g., "azure", "helm", "k8s")
2. **Package Description**: Clear description of package purpose and capabilities
3. **Package Domain**: Technology domain (cloud provider, container orchestration, database, etc.)
4. **Component Types**: Which component types will be included (activities, models, resources, etc.)

### Optional Information
- **Dependencies**: Required and optional package dependencies
- **Version**: Semantic version (defaults to 0.1.0)
- **Owner**: Package maintainer (defaults to "Team ClrSlate")
- **Color**: UI color coding (hex format)
- **Keywords**: Search and discovery keywords

## Package Structure Creation

Create the following standardized directory structure:

### Required Files
```
package-name/
├── metadata.yaml              # Package metadata and dependencies
└── README.md                  # Package documentation
```

### Component Directories (Include as needed)
```
├── activities/                # Activity definitions (.yaml files)
├── models/                    # ModelDefinition and CredentialDefinition files
├── resources/                 # Resource instances and configurations
│   ├── helmCharts/           # Helm chart configurations
│   └── helmReleases/         # Helm release configurations
├── configs/                   # Configuration files and mappings
├── pipelines/                 # Tekton pipeline definitions (.pipeline.yaml)
└── pipelineRefs/             # Tekton PipelineRef handler definitions
```

## Metadata.yaml Requirements

Generate metadata.yaml with the following structure:

```yaml
name: <package-name>                    # Required: kebab-case identifier
version: <semantic-version>             # Required: x.y.z format (default: 0.1.0)
title: <display-title>                  # Required: Human-readable name
description: <package-description>      # Required: Purpose and capabilities
owner: <owner-identifier>               # Required: Maintainer (default: Team ClrSlate)
icon: <icon-name>                       # Required: From predefined icon set

# Optional dependencies
optional_dependencies:                  # Packages that enhance functionality
  - <package-name>
required_dependencies:                  # Packages required for functionality
  - <package-name>

# Optional extended metadata
color: <color-value>                    # UI color coding (hex format)
keywords:                              # Search keywords
  - <keyword>
maintainers:                           # Maintainer details
  - name: <maintainer-name>
```

## Icon Selection Guidelines

Choose appropriate icon based on package type:

### Technology-Specific Icons
- **Cloud Providers**: Azure.Logo, Amazon.Aws, Google.Gcp
- **Container**: Kubernetes.Logo, Kubernetes.Helm, Brands.Istio
- **Databases**: Brands.MongoDb, Brands.PostgreSql, Brands.MySql, Brands.Redis
- **Version Control**: Brands.GitHub, Brands.GitLab, Brands.Git
- **Languages**: Brands.DotNet, Brands.Python, Brands.Java, Brands.Nodejs

### Generic Icons
- **General Purpose**: Generic.Database, Generic.Streaming, Generic.Messaging, Generic.Stack

## Dependency Management

### Required Dependencies
Use when package **cannot function** without dependency:
- Kubernetes operations require `k8s` package
- Azure operations require `azure` package

### Optional Dependencies
Use when package is **enhanced by** dependency:
- Monitoring capabilities with `observability` package
- DNS management with `externalDns` package

## README.md Template

Generate comprehensive README.md with:

```markdown
# <Package Title>

## Overview
<Brief description of package purpose and capabilities>

## Components
- **Activities**: <List key activities if present>
- **Models**: <List key models if present>
- **Resources**: <List key resources if present>

## Dependencies
### Required
<List required dependencies and why they're needed>

### Optional
<List optional dependencies and what they enable>

## Usage
<Basic usage examples and integration patterns>

## Configuration
<Configuration requirements and options>
```

## File Naming Conventions

Follow these naming patterns:

### Activities
- `create-<resource-type>.yaml` - Creation operations
- `delete-<resource-type>.yaml` - Deletion operations  
- `deploy-<component>.yaml` - Deployment operations
- `<action>-<target>.yaml` - General pattern

### Models
- `<resource-type>.yaml` - Primary resource models
- `credentials.yaml` - Credential definitions
- `<domain>-<type>.yaml` - Domain-specific models

### Resources
- `helmCharts/<chart-name>.yaml` - Helm chart configurations
- `helmReleases/<release-name>.yaml` - Helm release configurations

### Pipelines
- `<package>-<purpose>.pipeline.yaml` - Pipeline definitions
- `<action>-<target>.yaml` - PipelineRef handlers

## Validation Checklist

Ensure the created package meets these requirements:

- [ ] Package name uses kebab-case format
- [ ] metadata.yaml contains all required fields
- [ ] Icon selection matches package domain
- [ ] Dependencies are correctly categorized (required vs optional)
- [ ] README.md provides clear documentation
- [ ] Directory structure follows standard layout
- [ ] File names follow established conventions
- [ ] Version follows semantic versioning (x.y.z)

## Component Creation Integration

When creating package components, use the specialized prompts for each component type:

### Activities
- **Use**: `create-package-activity.prompt.md` for creating ClrSlate activities
- **Process**: Determine handler type (console vs tekton.pipelineRef) based on complexity
- **Integration**: Ensure activities align with package domain and naming conventions
- **Dependencies**: For complex operations, may require corresponding PipelineRef resources

### PipelineRef Resources  
- **Use**: `create-tekton-pipelineref.prompt.md` for creating Tekton PipelineRef resources
- **Process**: Define schema, parameter mapping, and secret handling
- **Integration**: Link with activities using tekton.pipelineRef handler
- **Requirements**: Requires Tekton cluster infrastructure

### Models and Resources
- **Create manually**: Based on package domain requirements
- **Follow naming**: Use established file naming conventions
- **Validate**: Ensure proper resource type definitions

## Example Workflow

1. **Gather Information**: Ask for package name, description, and domain
2. **Create Structure**: Generate directory layout based on component needs
3. **Generate Metadata**: Create metadata.yaml with appropriate fields
4. **Create Documentation**: Generate comprehensive README.md
5. **Create Components**: Use specialized prompts for activities and PipelineRef resources
6. **Validate**: Ensure all requirements are met and components integrate properly

Ask for the package name and domain if not provided, then follow this structured approach to create a complete, standards-compliant ClrSlate package.

When the user requests activities for the package, use the `create-package-activity.prompt.md` to ensure proper execution framework compliance and handler selection.

