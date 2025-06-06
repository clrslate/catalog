---
type: reference
category: foundation
complexity: basic
prerequisites: []
outputs: [platform-understanding, core-concepts, package-concepts]
lastUpdated: 2025-05-29
version: 1.1.0
---

# Platform Introduction

## Overview

ClrSlate is a workflow-driven platform engineering tool that simplifies backend automation through a dynamic type system and reusable components.

## Core Architecture

### Package System
Organized collection of related ClrSlate components for specific domains:
- **Package Structure**: Standardized directory layout with metadata
- **Dependencies**: Package relationships and requirements
- **Component Organization**: Models, activities, resources, and configurations

### Workflow Engine
DAG-based workflow execution for automating platform engineering processes.

### Dynamic Type System
Extensible type system enabling custom resource definitions:
- **ModelDefinition**: Resource schema definitions
- **Records**: Data instances conforming to ModelDefinition schemas
- **CredentialDefinition**: Structured secret management schemas
- **Secrets**: Platform integration credentials

### Activity Framework
Reusable execution blocks for workflow composition:
- **Activities**: Define workflow nodes with input schemas
- **ActivityHandlers**: Execution backends (Tekton, API calls, webhooks)
- **Mirrored Properties**: Dynamic value computation from inputs

## Resource Model

### Wrapper Structure
All ClrSlate resources follow Kubernetes-inspired wrapper pattern:

```yaml
apiVersion: <group-name>
kind: <resource-kind>
metadata:
    name: <unique-identifier>
    title: <display-name>
    # Optional: description, icon, color, tags, labels, annotations
spec:
    # Resource-specific configuration
```

### Resource Types
- **core.clrslate.io**: Platform core types (Activity, ModelDefinition)
- **records.clrslate.io**: Data instances
- **secrets.clrslate.io**: Credential definitions

## Decision Framework

### When to Create Packages
Create packages when:
- Grouping related components for a specific domain (e.g., Azure, Helm, Kubernetes)
- Building reusable component collections
- Managing dependencies between components
- Providing standardized deployment patterns

### When to Use ModelDefinition
Create ModelDefinition when:
- Defining custom resource schemas
- Extending platform capabilities
- Requiring typed data validation
- Building reusable resource templates

### When to Use Activities
Create Activities when:
- Implementing workflow operations
- Requiring parameterized execution
- Needing handler-agnostic logic
- Building reusable workflow components

### When to Use Records
Create Records when:
- Instantiating data from ModelDefinition schemas
- Storing configuration values
- Providing workflow input data
- Referencing resources in templates

## Handler Selection

### Tekton PipelineRef
Use for:
- Kubernetes-native execution
- Complex multi-step operations
- Container-based processing
- Pipeline orchestration

**Structure**:
- Schema: Input parameter definitions
- Mirrored: Derived property computation
- Pipeline: Tekton execution configuration

### Console Handler
Use for:
- Simple command execution
- Direct system integration
- Debugging and development
- Quick operational tasks

## Template Engine Integration

### Liquid Templates
Used in:
- Mirrored property value computation
- File content generation for handlers
- Dynamic parameter transformation
- Resource reference resolution

### Expression System
Records automatically convert properties with special formats:
- `resource` format → Resource reference expressions
- `secret` format → Secret reference expressions

## Platform Extension Pattern

### 1. Create Package Structure
```yaml
# metadata.yaml
name: my-platform
version: 1.0.0
title: My Platform
description: Custom platform integration
owner: Development Team
icon: Custom.Icon
```

### 2. Define Resource Schema
```yaml
apiVersion: core.clrslate.io
kind: ModelDefinition
metadata:
    name: azure-aks-cluster
spec:
    schema:
        properties:
            clusterName: {type: string}
            resourceGroup: {type: string}
            credential: {type: object, format: secret}
```

### 3. Create Resource Instances
```yaml
apiVersion: records.clrslate.io
kind: azure-aks-cluster
metadata:
    name: production-cluster
spec:
    clusterName: prod-aks
    resourceGroup: production-rg
    credential: azure-prod-creds
```

### 4. Build Activities
```yaml
apiVersion: core.clrslate.io
kind: Activity
metadata:
    name: deploy-to-aks
spec:
    inputs:
        properties:
            cluster: {type: object, format: resource}
    handler:
        type: tekton.pipelineRef
        # Handler configuration
```

### 5. Compose Workflows
Combine activities into DAG-based workflows for complete automation.

## Validation Rules

### Package Naming
- Package names must be unique and use kebab-case
- Use descriptive names reflecting primary domain or technology
- Examples: `azure`, `helm`, `k8s`, `istio`

### Resource Naming
- Names must be unique within their apiVersion/kind scope
- Use kebab-case for consistency
- Include descriptive prefixes for categorization

### Schema Design
- Define required properties explicitly
- Use appropriate type constraints
- Include descriptive titles and descriptions
- Specify format for special property types

### Handler Configuration
- Match handler capabilities to activity requirements
- Validate parameter mappings
- Ensure secret references are properly configured
- Test file content templates before deployment

## Success Patterns

### Package Organization
- Group related components by domain (Azure, Kubernetes, etc.)
- Use consistent naming conventions across packages
- Apply standardized labels for categorization
- Document dependencies between packages clearly

### Resource Organization
- Group related ModelDefinitions by domain
- Use consistent naming conventions
- Apply standardized labels for categorization
- Document dependencies between resources

### Activity Design
- Keep activities focused on single responsibilities
- Design for reusability across workflows
- Use mirrored properties for complex transformations
- Validate inputs thoroughly

### Workflow Composition
- Design workflows as directed acyclic graphs
- Handle dependencies explicitly
- Plan for error scenarios
- Include monitoring and observability

## Package Organization

### Standard Package Structure
```
package-name/
├── metadata.yaml              # Required: Package metadata and dependencies  
├── README.md                  # Required: Package documentation
├── activities/                # Activity definitions
├── models/                    # ModelDefinition and CredentialDefinition files
├── resources/                 # Resource instances and configurations
├── configs/                   # Configuration files and mappings
├── pipelines/                 # Tekton pipeline definitions
├── pipelineRefs/             # Tekton pipeline references
└── invocation-handlers/       # Custom handler definitions
```

### Package Metadata
Required metadata.yaml structure:
```yaml
name: azure                    # Unique package identifier
version: 0.1.0                # Semantic version
title: Azure                  # Human-readable display name
description: Azure cloud infrastructure resources
owner: Team ClrSlate          # Package maintainer
icon: Azure.Logo              # Visual representation

optional_dependencies:        # Packages that enhance functionality
  - k8s
required_dependencies: []     # Packages required for functionality
```

### Package Examples
- **azure**: Azure cloud infrastructure components
- **helm**: Helm chart management and deployment
- **k8s**: Kubernetes resource management
- **istio**: Service mesh configuration and deployment
