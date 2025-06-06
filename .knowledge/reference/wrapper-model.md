---
type: reference
category: core-concepts
complexity: intermediate
prerequisites: [platform-introduction, architecture-overview]
outputs: [data-model-understanding, schema-design-capability]
lastUpdated: 2025-05-28
version: 1.0.0
---

# Wrapper Model

## Overview

ClrSlate uses a Kubernetes-inspired wrapper model that provides consistent structure across all resource types. This unified approach enables standardized metadata handling, validation, and resource management.

## Universal Structure

Every ClrSlate resource follows this pattern:

```yaml
apiVersion: <group-name>    # Resource namespace and versioning
kind: <resource-kind>       # Resource type identifier
metadata:                   # Resource identification and organization
    name: <unique-name>     # Required: Unique identifier within scope
    title: <display-title>  # Required: Human-readable display name
    description: <text>     # Optional: Detailed description
    icon: <icon-name>       # Optional: Visual representation
    color: <color-value>    # Optional: UI color coding
    tags: [<tag1>, <tag2>]  # Optional: Categorization tags
    labels:                 # Optional: Key-value organization
        <key>: <value>
    annotations:            # Optional: Extended metadata
        <key>: <value>
spec:                       # Resource-specific configuration
    # Content varies by resource type
```

## Metadata System

### Required Fields
- **name**: Unique identifier within apiVersion/kind scope
- **title**: Human-readable display name for UI presentation

### Organizational Fields
- **description**: Detailed explanation of resource purpose
- **tags**: Array of categorization labels for filtering
- **labels**: Key-value pairs for systematic organization
- **annotations**: Extended metadata for tools and integrations

### Visual Fields
- **icon**: Predefined icon identifier for UI representation
- **color**: Color value for visual distinction

## Resource Scoping

### API Versioning
Resources are organized by apiVersion namespace:
- **core.clrslate.io**: Platform core types (ModelDefinition, Activity, SecretDefinition)
- **records.clrslate.io**: Data instances
- **secrets.clrslate.io**: Credential instances
- **tekton.clrslate.io**: Tekton handler configurations

### Kind Classification
The `kind` field identifies resource type within namespace:
- Platform-defined kinds: ModelDefinition, Activity, SecretDefinition
- User-defined kinds: Custom model names from ModelDefinitions

## Standardized Labels

ClrSlate recognizes these standardized label keys:

### Organizational Labels
- **owner**: Resource owner or maintainer identifier
- **package**: Package or module grouping
- **category**: Functional category classification

### Usage Patterns
```yaml
labels:
    owner: platform-team
    package: azure-integration
    category: infrastructure
```

## Icon System

The `metadata.icon` field supports predefined icons from a flat lookup system. Icon values must be selected from the following available options:

Azure.Logo, Azure.AKS, Azure.DevOps, Amazon.Aws, Kubernetes.Logo, Kubernetes.Helm, Brands.Istio, Brands.Nginx, Brands.MongoDb, Brands.PostgreSql, Brands.MySql, Brands.Redis, Brands.ElasticSearch, Brands.GitHub, Brands.GitLab, Brands.Git, Brands.Jenkins, Brands.VisualStudio, Brands.DotNet, Brands.Python, Brands.Java, Brands.Nodejs, Brands.Ruby, Generic.Database, Generic.Streaming, Generic.Messaging, Generic.Stack, Generic.Robot, Generic.AI

## Validation Framework

### Name Constraints
- Must be unique within apiVersion/kind scope
- Should use kebab-case convention
- Include domain prefixes for organization

### Metadata Validation
- Required fields must be present
- Icon values must match predefined set
- Labels and annotations follow key-value format

### Consistency Rules
- Related resources should use consistent naming patterns
- Icon selection should reflect resource purpose
- Labels should follow organizational standards

## Design Patterns

### Naming Conventions
```yaml
# ModelDefinition naming
name: domain.model.resource-type
# Example: azure.model.aks-cluster

# Record naming  
name: domain.records.model-type.instance-name
# Example: azure.records.aks-cluster.production
```

### Metadata Hierarchy
```yaml
metadata:
    name: azure-aks-cluster
    title: Azure AKS Cluster
    description: Azure Kubernetes Service cluster configuration
    icon: Azure.AKS
    color: "#0078D4"
    tags:
        - azure
        - kubernetes
        - infrastructure
    labels:
        owner: platform-team
        package: azure-integration
        category: compute
    annotations:
        documentation: https://docs.azure.com/aks
        source-repo: https://github.com/org/azure-models
```

### Resource Relationships
ClrSlate wrapper model enables:
- **Reference Resolution**: Resources can reference others by name
- **Hierarchical Organization**: Labels and tags create logical groupings
- **Template Integration**: Metadata becomes available in template contexts
- **Audit Trailing**: Wrapper provides complete change tracking

## Integration Points

### Template Engine
Wrapper metadata is accessible in templates:
- `{{metadata.name}}` - Resource identifier
- `{{metadata.title}}` - Display name
- `{{metadata.labels.owner}}` - Label values
- `{{spec.propertyName}}` - Specification properties

### Resource Discovery
Wrapper model enables systematic resource discovery:
- Filter by apiVersion and kind
- Search by labels and tags
- Query by metadata fields
- Traverse resource relationships

### Validation Integration
Wrapper structure provides validation hooks:
- Schema validation for spec contents
- Metadata constraint checking
- Reference integrity verification
- Naming convention enforcement

## AI Agent Guidelines

### Resource Creation
When creating ClrSlate resources:
1. **Select appropriate apiVersion** based on resource type
2. **Choose descriptive kind** that reflects resource purpose
3. **Design meaningful metadata** for discoverability
4. **Follow naming conventions** for consistency
5. **Include relevant labels** for organization

### Schema Design
For ModelDefinition and SecretDefinition:
1. **Use wrapper model** as foundation
2. **Define clear spec structure** for resource-specific data
3. **Include comprehensive metadata** for documentation
4. **Apply validation constraints** appropriate to purpose
5. **Consider integration patterns** with other resources

### Resource Organization
Organize resources using wrapper model features:
1. **Group by apiVersion** for logical separation
2. **Use labels consistently** across related resources
3. **Apply tags systematically** for categorization
4. **Include annotations** for extended metadata
5. **Design hierarchical naming** for clear relationships
