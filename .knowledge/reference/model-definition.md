---
type: reference
category: core-concepts
complexity: advanced
prerequisites: [wrapper-model]
outputs: [schema-definition-capability, model-relationship-understanding]
lastUpdated: 2025-05-28
version: 1.0.0
---

# ModelDefinition

## Overview

ModelDefinition defines extensible resource schemas within ClrSlate's dynamic type system. It enables custom resource types with validation, relationships, and computed properties for platform integration.

## Structure

```yaml
apiVersion: core.clrslate.io
kind: ModelDefinition
metadata:
    name: <unique-model-name>           # Required: Model identifier
    title: <display-title>             # Required: Human-readable name
    description: <description>         # Optional: Model purpose
    icon: <icon-name>                  # Optional: Visual representation
    color: <color-value>               # Optional: UI color
    tags: [<domain-tags>]              # Optional: Categorization
    labels:                            # Optional: Organization
        category: <category-name>
        package: <package-name>
spec:
    schema:                            # Required: Input validation schema
        properties:                    # Property definitions
            <property-name>:
                type: <type>           # JSON Schema type
                title: <title>        # Required: Display name
                description: <desc>    # Property purpose
                # Validation constraints
        required: [<required-props>]   # Required property list
    mirrored:                          # Optional: Computed properties
        <property-name>:
            type: <type>               # Property type
            value: <template>          # Value computation template
```

## Schema System

### Property Types
ModelDefinition supports standard JSON Schema types:

**Primitive Types**:
- `string`: Text values with format validation
- `number`: Numeric values with range constraints
- `integer`: Whole numbers with bounds
- `boolean`: True/false values

**Complex Types**:
- `object`: Structured data with nested properties
- `array`: Collections with item validation

### Special Object Formats

**Resource References** (`format: resource`):
```yaml
cluster:
    type: object
    format: resource
    title: Kubernetes Cluster
    description: Target cluster for deployment
    specifications:
        type: azure.model.aks-cluster    # Target ModelDefinition
```

**Secret References** (`format: secret`):
```yaml
credentials:
    type: object
    format: secret
    title: Azure Credentials
    description: Authentication credentials
    specifications:
        type: azure.secret.service-principal
```

### Validation Constraints

**String Validation**:
```yaml
name:
    type: string
    pattern: "^[a-z][a-z0-9-]*$"      # Regex pattern
    minLength: 3                       # Minimum length
    maxLength: 63                      # Maximum length
    format: hostname                   # Format validation
```

**Numeric Validation**:
```yaml
port:
    type: integer
    minimum: 1                         # Lower bound
    maximum: 65535                     # Upper bound
```

**Array Validation**:
```yaml
tags:
    type: array
    items:
        type: string                   # Item type
    minItems: 1                        # Minimum count
    maxItems: 10                       # Maximum count
    uniqueItems: true                  # Uniqueness requirement
```

**Enum Constraints**:
```yaml
environment:
    type: string
    enum: [development, staging, production]
    description: Deployment environment
```

## Mirrored Properties

Mirrored properties provide computed values derived from schema inputs:

### Value Templates
```yaml
mirrored:
    command:
        type: string
        value: |
            helm upgrade --install {{inputs.name}} {{inputs.chart.name}} 
            --repo {{inputs.chart.repository}} 
            --version {{inputs.chart.version}} 
            --namespace {{inputs.namespace}}
```

### Resource Projections
```yaml
mirrored:
    defaultChart:
        type: object
        format: resource
        value: mongo.records.helmChart.default-mongodb
```

### Templatized References
```yaml
mirrored:
    clusterCredentials:
        type: object
        format: secret
        value: '{{inputs.resourceGroup.subscription.credentials._name}}'
```

## Template Engine Integration

### Property Access Patterns
- **Direct Properties**: `{{inputs.propertyName}}`
- **Nested Objects**: `{{inputs.object.property}}`
- **Resource References**: `{{inputs.cluster.name}}`
- **Array Elements**: `{{inputs.tags[0]}}`

### Reference Resolution
When properties use `format: resource` or `format: secret`:
1. **Property Value**: Contains reference to target Record
2. **Template Access**: Properties of referenced Record become available
3. **Dot Notation**: Access nested properties using standard syntax
4. **Type Safety**: Referenced properties maintain their defined types

### Complex Reference Paths
```yaml
# Schema defines nested resource reference
resourceGroup:
    type: object
    format: resource
    specifications:
        type: azure.model.resource-group

# Mirrored property traverses reference chain
mirrored:
    subscriptionCredentials:
        type: object
        format: secret
        value: '{{inputs.resourceGroup.subscription.credentials._name}}'
```

## Model Relationships

### One-to-One References
```yaml
# HelmRelease references single HelmChart
chart:
    type: object
    format: resource
    specifications:
        type: helm.model.helmChart
    required: true
```

### One-to-Many References
```yaml
# Cluster references multiple NodePools
nodePools:
    type: array
    items:
        type: object
        format: resource
        specifications:
            type: azure.model.node-pool
```

### Hierarchical Relationships
```yaml
# KeyVault references ResourceGroup which references Subscription
resourceGroup:
    type: object
    format: resource
    specifications:
        type: azure.model.resource-group

# Access hierarchical properties in templates
mirrored:
    subscriptionId:
        type: string
        value: '{{inputs.resourceGroup.subscription.id}}'
```

## Design Patterns

### Domain Modeling
```yaml
# Base infrastructure model
apiVersion: core.clrslate.io
kind: ModelDefinition
metadata:
    name: azure.model.resource-group
    title: Azure Resource Group
    labels:
        category: infrastructure
        package: azure-core
spec:
    schema:
        properties:
            name:
                type: string
                pattern: "^[a-zA-Z0-9-_]+$"
            location:
                type: string
                enum: [eastus, westus, westeurope]
            subscription:
                type: object
                format: resource
                specifications:
                    type: azure.model.subscription
        required: [name, location, subscription]
```

### Service Modeling
```yaml
# Application service model
apiVersion: core.clrslate.io
kind: ModelDefinition
metadata:
    name: app.model.microservice
    title: Microservice
    labels:
        category: application
        package: app-platform
spec:
    schema:
        properties:
            name:
                type: string
                description: Service name
            version:
                type: string
                pattern: "^v\\d+\\.\\d+\\.\\d+$"
            cluster:
                type: object
                format: resource
                specifications:
                    type: kubernetes.model.cluster
            database:
                type: object
                format: resource
                specifications:
                    type: database.model.instance
        required: [name, version, cluster]
    mirrored:
        namespace:
            type: string
            value: '{{inputs.name}}-{{inputs.version}}'
```

## Validation Framework

### Schema Validation Rules
1. **Required Properties**: Must be present in Records
2. **Type Constraints**: Values must match defined types
3. **Format Validation**: Special formats (email, uri, date-time) enforced
4. **Range Constraints**: Numeric bounds and string lengths validated
5. **Pattern Matching**: Regex patterns applied to strings
6. **Enum Validation**: Values must be from allowed set

### Reference Validation
1. **Target Existence**: Referenced ModelDefinitions must exist
2. **Type Compatibility**: Referenced Records must match specifications.type
3. **Circular Dependency**: Prevent recursive reference chains
4. **Access Permissions**: Validate read access to referenced resources

### Custom Validation Rules
```yaml
# Example: Validate Kubernetes resource naming
name:
    type: string
    pattern: "^[a-z0-9]([-a-z0-9]*[a-z0-9])?$"
    maxLength: 63
    description: Must follow Kubernetes naming conventions
```

## AI Agent Guidelines

### Schema Design Principles
1. **Single Responsibility**: Model one logical concept per definition
2. **Clear Relationships**: Use explicit resource references
3. **Validation Completeness**: Include all necessary constraints
4. **Template Compatibility**: Design for template engine integration
5. **Evolution Planning**: Allow for schema extension over time

### Property Design
1. **Descriptive Names**: Use clear, unambiguous property names
2. **Appropriate Types**: Choose types that match data semantics
3. **Validation Logic**: Include constraints that prevent invalid states
4. **Reference Semantics**: Use resource/secret formats for relationships
5. **Default Values**: Provide sensible defaults where appropriate

### Mirrored Property Usage
1. **Computed Values**: Generate derived data from inputs
2. **Command Templates**: Build execution commands dynamically
3. **Reference Simplification**: Expose complex reference paths simply
4. **Default Resources**: Provide standard resource references
5. **Value Transformation**: Convert input formats for specific uses

### Model Organization
1. **Domain Grouping**: Group related models by functional domain
2. **Naming Consistency**: Use consistent naming patterns across models
3. **Package Structure**: Organize models into logical packages
4. **Dependency Management**: Design clear dependency hierarchies
5. **Version Strategy**: Plan for model evolution and compatibility
