
````prompt
---
mode: 'agent'
description: 'Create a concise, standards-compliant ClrSlate ModelDefinition.'
---

# ClrSlate ModelDefinition Quick Guide

## Required Fields

| Field                   | Required | Type   | Notes/Allowed Values                  |
|-------------------------|----------|--------|---------------------------------------|
| metadata.name           | Yes      | string | `<domain>.model.<resource-type>`      |
| metadata.title          | Yes      | string | Human-readable                        |
| spec.schema             | Yes      | object | JSON schema                           |
| spec.schema.required    | Yes      | array  | List of required property names       |
| spec.schema.properties  | Yes      | object | Each: type, description, validation   |
| spec.mirrored           | Optional | object | Each: type, value (template)          |

## ModelDefinition Template

```yaml
apiVersion: core.clrslate.io
kind: ModelDefinition
metadata:
  name: <domain>.model.<resource-type>
  title: <display-title>
  description: <model-purpose>
spec:
  schema:
    type: object
    required:
      - <required-property>
    properties:
      <property-name>:
        type: <property-type>
        description: <property-purpose>
        # validation (pattern, enum, minLength, etc.)
  mirrored:
    <computed-property>:
      type: <property-type>
      value: "<liquid-template>"
```

## Validation Checklist

- [ ] Model name uses `<domain>.model.<resource-type>`
- [ ] All required fields present
- [ ] All property types and validation correct
- [ ] All references use correct format/specifications
- [ ] All mirrored properties use `type:` and `value:`
- [ ] File is named `<resource-type>.yaml` and placed in `models/`

## Common Mistakes

- Do NOT use `template:` in mirrored properties—use `type:` and `value:`.
- Do NOT omit required fields in metadata or schema.

## Reference Examples

### String with validation
```yaml
name:
  type: string
  description: Resource name
  pattern: "^[a-z0-9-]+$"
  minLength: 1
  maxLength: 64
```

### Integer with bounds
```yaml
port:
  type: integer
  minimum: 1
  maximum: 65535
```

### Enum
```yaml
environment:
  type: string
  enum: [development, staging, production]
```

### Array
```yaml
tags:
  type: array
  items:
    type: string
  minItems: 1
  uniqueItems: true
```

### Resource reference
```yaml
resourceGroup:
  type: object
  format: resource
  specifications:
    type: azure.model.resource-group
```

### Secret reference
```yaml
credentials:
  type: object
  format: secret
  specifications:
    type: azure.credential.service-principal
```

### Mirrored property
```yaml
mirrored:
  resourceId:
    type: string
    value: "/subscriptions/{{inputs.subscription.subscriptionId}}/resourceGroups/{{inputs.name}}"
```

- [ ] Model name uses `<domain>.model.<resource-type>`
- [ ] All required fields present
- [ ] All property types and validation correct
- [ ] All references use correct format/specifications
- [ ] All mirrored properties use `type:` and `value:`
- [ ] File is named `<resource-type>.yaml` and placed in `models/`

## Common Mistakes

- Do NOT use `template:` in mirrored properties—use `type:` and `value:`.
- Do NOT omit required fields in metadata or schema.

## Reference Examples

# String with validation
```yaml
name:
  type: string
  description: Resource name
  pattern: "^[a-z0-9-]+$"
  minLength: 1
  maxLength: 64
```

# Integer with bounds
```yaml
port:
  type: integer
  minimum: 1
  maximum: 65535
```

# Enum
```yaml
environment:
  type: string
  enum: [development, staging, production]
```

# Array
```yaml
tags:
  type: array
  items:
    type: string
  minItems: 1
  uniqueItems: true
```

# Resource reference
```yaml
resourceGroup:
  type: object
  format: resource
  specifications:
    type: azure.model.resource-group
```

# Mirrored property
```yaml
mirrored:
  resourceId:
    type: string
    value: "/subscriptions/{{inputs.subscription.subscriptionId}}/resourceGroups/{{inputs.name}}"
```

## Schema Property Types

### Primitive Types
```yaml
# String with validation
name:
  type: string
  description: Resource name
  pattern: "^[a-z0-9-]+$"           # Optional regex pattern
  minLength: 1                       # Optional length constraints
  maxLength: 64

# String with format validation
email:
  type: string
  format: email                      # Built-in format validation

# Integer with bounds
port:
  type: integer
  description: Port number
  minimum: 1                         # Range constraints
  maximum: 65535

# Number with constraints
cpu:
  type: number
  description: CPU allocation
  minimum: 0.1
  multipleOf: 0.1                   # Increment validation

# Boolean
enabled:
  type: boolean
  description: Feature enabled flag
  default: true                      # Default value
```

### Complex Types
```yaml
# Object properties
metadata:
  type: object
  description: Resource metadata
  properties:
    environment:
      type: string
      enum: ["dev", "staging", "prod"]
    tags:
      type: array
      items:
        type: string
      uniqueItems: true

# Array with validation
nodeCount:
  type: array
  description: Node configuration
  items:
    type: object
    properties:
      size:
        type: string
      count:
        type: integer
        minimum: 1
```

### Special Reference Types

#### Resource References
```yaml
# Reference to another ClrSlate resource
resourceGroup:
  type: object
  format: resource                   # Special format for resource references
  description: Target resource group
  specifications:
    type: azure.model.resource-group # Expected model type

# Optional resource reference
cluster:
  type: object
  format: resource
  description: Kubernetes cluster (optional)
  specifications:
    type: k8s.model.cluster
```

#### Secret References
```yaml
# Reference to credential definition
credentials:
  type: object
  format: secret                     # Special format for secret references
  description: Azure service principal credentials
  specifications:
    type: azure.credential.service-principal # Expected credential type

# Optional secret reference
databasePassword:
  type: object
  format: secret
  description: Database password (optional)
  specifications:
    type: database.credential.password
```

## Mirrored Properties (Computed Values)

### Value Transformation
```yaml
mirrored:
  # Simple property transformation
  resourceName:
    type: string
    value: "{{inputs.name | downcase}}-{{inputs.environment}}"
  
  # Conditional logic
  defaultTags:
    type: string
    value: |
      {%- assign tags = inputs.tags | default: [] -%}
      {%- unless tags contains inputs.environment -%}
        {%- assign tags = tags | append: inputs.environment -%}
      {%- endunless -%}
      {{tags | join: ","}}
```

### Resource Property Access
```yaml
mirrored:
  # Access properties from referenced resources
  subscriptionId:
    type: string
    value: "{{inputs.resourceGroup.subscription.subscriptionId}}"
  
  # Combine resource properties
  clusterCredentials:
    type: string
    value: "{{inputs.cluster.credentials}}"
  
  # Default values from references
  location:
    type: string
    value: "{{inputs.resourceGroup.location | default: 'eastus'}}"
```

### Complex Computed Properties
```yaml
mirrored:
  # Generate configuration objects
  deploymentConfig:
    type: object
    value: |
      {
        "location": "{{inputs.resourceGroup.location}}",
        "environment": "{{inputs.environment}}",
        "resourceGroup": "{{inputs.resourceGroup.name}}",
        "subscriptionId": "{{inputs.resourceGroup.subscription.subscriptionId}}"
      }
  
  # Generate commands or scripts
  createCommand:
    type: string
    value: |
      az group create \
        --name {{inputs.name}} \
        --location {{inputs.resourceGroup.location}} \
        --subscription {{inputs.resourceGroup.subscription.subscriptionId}}
```

## Validation Patterns

### String Validation
```yaml
# Pattern matching
resourceName:
  type: string
  pattern: "^[a-z][a-z0-9-]*[a-z0-9]$"
  description: Must start with letter, contain only lowercase letters, numbers, and hyphens

# Format validation
endpoint:
  type: string
  format: uri
  description: Service endpoint URL

# Length constraints
description:
  type: string
  minLength: 10
  maxLength: 500
```

### Numeric Validation
```yaml
# Range constraints
instanceCount:
  type: integer
  minimum: 1
  maximum: 100
  description: Number of instances to deploy

# Multiple validation
memory:
  type: number
  multipleOf: 0.5
  minimum: 0.5
  description: Memory allocation in GB (increments of 0.5)
```

### Enum Validation
```yaml
# Predefined values
tier:
  type: string
  enum: ["Basic", "Standard", "Premium"]
  description: Service tier selection

# Multiple enum values
regions:
  type: array
  items:
    type: string
    enum: ["eastus", "westus", "centralus", "westeurope"]
  description: Deployment regions
```

## Design Patterns

### Infrastructure Model Pattern
```yaml
# Base infrastructure resource
apiVersion: core.clrslate.io
kind: ModelDefinition
metadata:
  name: azure.model.resource-group
  title: Azure Resource Group
  description: Azure resource group for organizing related resources
spec:
  schema:
    type: object
    required: ["name", "subscription"]
    properties:
      name:
        type: string
        description: Resource group name
        pattern: "^[a-zA-Z0-9-_]+$"
      location:
        type: string
        description: Azure region
        enum: ["eastus", "westus", "centralus", "westeurope"]
        default: "eastus"
      subscription:
        type: object
        format: resource
        description: Azure subscription
        specifications:
          type: azure.model.subscription
      tags:
        type: object
        description: Resource tags
        additionalProperties:
          type: string
  mirrored:
    resourceId:
      type: string
      value: "/subscriptions/{{inputs.subscription.subscriptionId}}/resourceGroups/{{inputs.name}}"
```

### Service Model Pattern
```yaml
# Application service resource
apiVersion: core.clrslate.io
kind: ModelDefinition
metadata:
  name: k8s.model.deployment
  title: Kubernetes Deployment
  description: Kubernetes deployment configuration for applications
spec:
  schema:
    type: object
    required: ["name", "image", "namespace"]
    properties:
      name:
        type: string
        description: Deployment name
        pattern: "^[a-z0-9-]+$"
      image:
        type: string
        description: Container image
        format: docker-image
      namespace:
        type: object
        format: resource
        description: Target namespace
        specifications:
          type: k8s.model.namespace
      replicas:
        type: integer
        description: Number of replicas
        minimum: 1
        default: 1
      resources:
        type: object
        description: Resource requirements
        properties:
          cpu:
            type: string
            pattern: "^[0-9]+m?$"
          memory:
            type: string
            pattern: "^[0-9]+[MGT]i?$"
  mirrored:
    fullImage:
      template: "{{inputs.image}}:{{inputs.tag | default: 'latest'}}"
    deploymentYaml:
      template: |
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: {{inputs.name}}
          namespace: {{inputs.namespace.name}}
        spec:
          replicas: {{inputs.replicas}}
          selector:
            matchLabels:
              app: {{inputs.name}}
          template:
            metadata:
              labels:
                app: {{inputs.name}}
            spec:
              containers:
              - name: {{inputs.name}}
                image: {{inputs.fullImage}}
```

## Validation Checklist

Ensure the created ModelDefinition meets these requirements:

- [ ] Model name follows qualified naming pattern (`domain.model.resource-type`)
- [ ] Schema includes all required properties with appropriate validation
- [ ] Property types are correctly specified (string, number, integer, boolean, object, array)
- [ ] Resource references use `format: resource` with proper specifications
- [ ] Secret references use `format: secret` with credential type specifications
- [ ] Mirrored properties use valid Liquid template syntax
- [ ] Enum values are comprehensive and appropriate for the domain
- [ ] Pattern validation uses correct regex syntax
- [ ] Range constraints (minimum, maximum) are logical
- [ ] Default values are sensible and safe
- [ ] Description fields are clear and informative

## File Naming Convention

Save ModelDefinition files using the pattern:
- **File Name**: `<resource-type>.yaml` (e.g., `resource-group.yaml`, `deployment.yaml`)
- **Location**: Place in the package's `models/` directory
- **Grouping**: Group related models in the same file if they form a logical unit

## Integration with Records

Remember that ModelDefinition serves as a schema for Records. The properties defined in the schema will be:
1. **Validated** when Records are created
2. **Available** in template expressions as `{{inputs.propertyName}}`
3. **Referenced** by other resources through resource reference format
4. **Computed** through mirrored properties for derived values

When creating the ModelDefinition, consider how Records will use these properties and what template expressions will need access to them.

Ask for the model name and domain if not provided, then follow this structured approach to create a comprehensive, standards-compliant ClrSlate ModelDefinition with proper validation and computed properties.
````
