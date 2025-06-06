# Execution Framework Examples

This document provides an overview of the ClrSlate execution framework examples, organized by handler type for better maintainability and discoverability.

## Example Organization

The execution framework examples are organized into handler-specific directories:

- **[Console Handler Examples](invocation-handlers/console/examples.md)** - Display and debugging activities
- **[Tekton PipelineRef Handler Examples](invocation-handlers/tekton-pipelineref/examples.md)** - Pipeline-based execution activities

Each handler directory contains complete, working examples demonstrating:
- Basic usage patterns
- Advanced configuration scenarios
- Expression system usage
- Validation examples
- Performance considerations

## Quick Start Examples

### Console Handler Quick Start

Simple debug activity for immediate testing:

```yaml
apiVersion: core.clrslate.io
kind: Activity
metadata:
  name: debug.activity.quickTest
  title: Quick Debug Test
  description: Simple console output for testing
spec:
  inputs:
    properties:
      message:
        type: string
        default: "Hello ClrSlate!"
  handler:
    type: console
    properties:
      output: "🔍 Debug: {{inputs.message}}"
```

### Tekton PipelineRef Quick Start

Basic pipeline execution activity:

```yaml
apiVersion: core.clrslate.io
kind: Activity
metadata:
  name: deploy.activity.quickDeploy
  title: Quick Deploy Test
  description: Simple deployment pipeline test
spec:
  inputs:
    properties:
      appName:
        type: string
        default: "test-app"
      credentials:
        type: object
        format: secret
        specifications:
          type: azure.secret.clientCredentials
  handler:
    type: tekton.pipelineRef
    properties:
      pipeline: azure.pipelineRef.simple-deploy
      inputs:
        appName: "{{inputs.appName}}"
        credentials: "{{inputs.credentials._name}}"
```

## Expression System Examples

### Resource Expression Access

When using resource expressions in activities:

```yaml
# In activity input
inputs:
  properties:
    cluster:
      type: object
      format: resource
      specifications:
        type: azure.model.aks

# Expression usage in handlers
handler:
  properties:
    inputs:
      clusterName: "{{inputs.cluster._name}}"          # Resource name
      resourceGroup: "{{inputs.cluster.resourceGroup.name}}"  # Spec property
      apiVersion: "{{inputs.cluster._apiVersion}}"     # API version
      metadata: "{{inputs.cluster._metadata.labels}}"  # Metadata access
```

### Secret Expression Access

When using secret expressions for mounting:

```yaml
# In activity input
inputs:
  properties:
    credentials:
      type: object
      format: secret
      specifications:
        type: azure.secret.clientCredentials

# Expression usage (must use ._name for mounting)
handler:
  properties:
    inputs:
      credentials: "{{inputs.credentials._name}}"  # Required for secret mounting
```

### Mirrored Property Access

When using computed properties:

```yaml
# In mirrored section
mirrored:
  fullCommand:
    type: string
    value: |
      echo "Deploying {{inputs.appName}} to {{inputs.namespace}}"
      kubectl apply -f /manifests/

# Usage in handler configuration
handler:
  properties:
    computedValue: "{{inputs.fullCommand}}"
```

## Validation Examples

### Activity Validation Success

```yaml
# This activity passes all validation rules
apiVersion: core.clrslate.io
kind: Activity
metadata:
  name: valid.activity.example  # ✅ Follows naming pattern
spec:
  inputs:
    properties:
      requiredInput:
        type: string
    required:
      - requiredInput  # ✅ Required input defined
  handler:
    type: console  # ✅ Supported handler type
    properties:
      output: "Value: {{inputs.requiredInput}}"  # ✅ Valid expression
```

### Common Validation Failures

```yaml
# Invalid naming pattern
metadata:
  name: InvalidName  # ❌ Should be: domain.category.name

# Missing required input mapping
spec:
  inputs:
    required: [requiredInput]
  handler:
    type: tekton.pipelineRef
    properties:
      inputs: {}  # ❌ Missing required input mapping

# Invalid secret expression
handler:
  properties:
    inputs:
      secret: "{{inputs.credentials}}"  # ❌ Should be: {{inputs.credentials._name}}
```

## Performance Considerations

### Resource Resolution Timing

```yaml
# Resource expressions are resolved once at activity start
# Cached for entire execution lifecycle
resources:
  cluster: "{{inputs.cluster._name}}"      # Resolved once
  credentials: "{{inputs.credentials._name}}"  # Resolved once

# Multiple references use cached values
handler:
  properties:
    inputs:
      clusterName: "{{inputs.cluster._name}}"     # Uses cached value
      clusterRegion: "{{inputs.cluster.region}}"  # Uses cached value
```

### Handler-Specific Performance

#### Console Handler Performance
- **Output Size**: Keep console output concise for performance
- **Expression Complexity**: Pre-compute complex expressions in mirrored properties
- **Template Logic**: Minimize conditional logic in output templates

See: [Console Performance Examples](invocation-handlers/console/examples.md#performance-considerations)

#### Tekton PipelineRef Performance
- **Secret Mounting**: Each mounted secret adds ~1-2 seconds to pipeline startup
- **Workspace Usage**: Use shared workspaces efficiently
- **Pipeline Complexity**: Prefer single-step tasks for better performance

See: [Tekton Performance Examples](invocation-handlers/tekton-pipelineref/examples.md#performance-considerations)

## Handler-Specific Examples

### Console Handler Examples
Complete examples for console-based activities including:
- Debug and validation displays
- Multi-resource status reports
- Configuration verification
- Error reporting with context
- Progress indicators and workflows

**📖 [View Console Handler Examples](invocation-handlers/console/examples.md)**

### Tekton PipelineRef Handler Examples
Complete examples for pipeline-based activities including:
- PipelineRef resource definitions
- Activity configurations with pipeline execution
- Corresponding Tekton pipeline specifications
- Advanced patterns like multi-stage deployments
- Conditional execution workflows

**📖 [View Tekton PipelineRef Handler Examples](invocation-handlers/tekton-pipelineref/examples.md)**

## Integration Examples

### Multi-Handler Workflow

Example showing console validation followed by pipeline execution:

```yaml
# Step 1: Console validation activity
apiVersion: core.clrslate.io
kind: Activity
metadata:
  name: workflow.activity.validateConfig
spec:
  handler:
    type: console
    properties:
      output: |
        🔍 Validating configuration for {{inputs.appName}}
        ✅ Configuration valid, proceeding to deployment

---
# Step 2: Pipeline execution activity  
apiVersion: core.clrslate.io
kind: Activity
metadata:
  name: workflow.activity.deployApp
spec:
  handler:
    type: tekton.pipelineRef
    properties:
      pipeline: azure.pipelineRef.app-deploy
      inputs:
        appName: "{{inputs.appName}}"
```

## Best Practices

### Expression Usage
1. **Resource Access**: Always use `._name` for resource references in pipeline contexts
2. **Secret Mounting**: Use `._name` pattern for secret mounting in Tekton handlers
3. **Property Access**: Use dot notation for accessing nested properties
4. **Validation**: Include input validation in activity schemas

### Handler Selection
1. **Console**: Use for validation, debugging, and status displays
2. **Tekton PipelineRef**: Use for deployment, build, and infrastructure operations
3. **Mixed Workflows**: Combine handlers for comprehensive automation workflows

### Performance Optimization
1. **Pre-compute**: Use mirrored properties for complex expressions
2. **Cache**: Leverage resource expression caching
3. **Minimize**: Keep handler configurations concise
4. **Validate**: Test expressions with debug console activities

## Related Documentation

- **[Execution Framework](execution-framework.md)** - Core concepts and architecture
- **[Console Handler](invocation-handlers/console/console.md)** - Console handler specification
- **[Tekton PipelineRef Handler](invocation-handlers/tekton-pipelineref/tekton-pipelineref.md)** - Pipeline handler specification
- **[Activity Specification](../activity-specification.md)** - Activity definition standards

## Contributing Examples

When adding new examples:

1. **Location**: Add to appropriate handler-specific examples file
2. **Structure**: Follow existing example patterns and formatting
3. **Documentation**: Include clear descriptions and use cases
4. **Validation**: Ensure examples pass platform validation
5. **Cross-reference**: Update this overview document with new patterns

For complex examples spanning multiple handlers, consider creating integration examples in this overview document.

```yaml
# In activity input
inputs:
  properties:
    cluster:
      type: object
      format: resource
      specifications:
        type: azure.model.aks

# Expression usage in handler
handler:
  type: tekton.pipelineRef
  properties:
    inputs:
      clusterName: "{{inputs.cluster._name}}"          # Resource name
      resourceGroup: "{{inputs.cluster.resourceGroup.name}}"  # Spec property
      apiVersion: "{{inputs.cluster._apiVersion}}"     # API version
      metadata: "{{inputs.cluster._metadata.labels}}"  # Metadata access
```

### Secret Expression Access

When using secret expressions for mounting:

```yaml
# In activity input
inputs:
  properties:
    credentials:
      type: object
      format: secret
      specifications:
        type: azure.secret.clientCredentials

# Expression usage in handler (must use ._name for mounting)
handler:
  type: tekton.pipelineRef
  properties:
    inputs:
      credentials: "{{inputs.credentials._name}}"  # Required for secret mounting
```

### Mirrored Property Access

When using computed properties:

```yaml
# In PipelineRef mirrored section
mirrored:
  fullCommand:
    type: string
    value: |
      echo "Deploying {{inputs.appName}} to {{inputs.namespace}}"
      kubectl apply -f /manifests/

# Usage in pipeline configuration
pipeline:
  files:
    deploy-script.sh: |
      #!/bin/bash
      {{inputs.fullCommand}}
```

## Validation Examples

### Activity Validation Success

```yaml
# This activity passes all validation rules
apiVersion: core.clrslate.io
kind: Activity
metadata:
  name: valid.activity.example  # ✅ Follows naming pattern
spec:
  inputs:
    properties:
      requiredInput:
        type: string
    required:
      - requiredInput  # ✅ Required input defined
  handler:
    type: console  # ✅ Supported handler type
    properties:
      output: "Value: {{inputs.requiredInput}}"  # ✅ Valid expression
```

### Activity Validation Failure Examples

```yaml
# Invalid naming pattern
metadata:
  name: InvalidName  # ❌ Should be: domain.category.name

# Missing required input mapping
spec:
  inputs:
    required: [requiredInput]
  handler:
    type: tekton.pipelineRef
    properties:
      inputs: {}  # ❌ Missing required input mapping

# Invalid secret expression
handler:
  properties:
    inputs:
      secret: "{{inputs.credentials}}"  # ❌ Should be: {{inputs.credentials._name}}
```

## Performance Considerations

### Resource Resolution Timing

```yaml
# Resource expressions are resolved once at activity start
# Cached for entire execution lifecycle
resources:
  cluster: "{{inputs.cluster._name}}"      # Resolved once
  credentials: "{{inputs.credentials._name}}"  # Resolved once

# Multiple references use cached values
handler:
  properties:
    inputs:
      clusterName: "{{inputs.cluster._name}}"     # Uses cached value
      clusterRegion: "{{inputs.cluster.region}}"  # Uses cached value
```

### Secret Mounting Performance

```yaml
# Secret mounting adds overhead to pipeline startup
# Minimize number of mounted secrets
spec:
  schema:
    properties:
      azureCreds:
        specifications:
          mount: true  # Adds ~2-3 seconds to pipeline startup
      gitCreds:
        specifications:
          mount: true  # Additional ~1-2 seconds per secret
```

These examples demonstrate the complete ClrSlate execution framework as documented in the platform specifications, including all features and constraints mentioned in the source documentation.
