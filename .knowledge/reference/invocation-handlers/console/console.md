---
type: reference
category: execution
subcategory: handlers
complexity: beginner
prerequisites: [execution-framework]
outputs: [console-activities, debug-workflows]
lastUpdated: 2025-05-28
version: 1.0.0
---

# Console Handler

The Console handler provides simple output generation capabilities for displaying information, debugging, and validation purposes within the ClrSlate execution framework.

## Overview

The Console handler is designed for activities that need to display formatted text output without triggering external processes. It's ideal for debugging, validation displays, configuration summaries, and user notifications.

## Examples

📖 **[Complete Console Handler Examples](examples.md)**

The examples include:
- Basic debug activities with configuration display
- Multi-resource status reports
- Input validation and verification workflows
- Error reporting with context and suggestions
- Expression system usage patterns
- Performance optimization techniques

## Handler Configuration

### Basic Pattern
```yaml
handler:
  type: console
  properties:
    output: <templated-content>
```

### Complete Configuration
```yaml
handler:
  type: console
  properties:
    output: |
      Deploying application: {{inputs.appName}}
      Target namespace: {{inputs.namespace}}
      Cluster: {{inputs.cluster._name}}
      Status: Ready for deployment
```

## When to Use

The Console handler is optimal for:

- **Debugging and validation** activities that need to verify input values
- **Status reporting** and logging for workflow visibility
- **Configuration verification** to display resolved configurations
- **Development and testing** workflows requiring output inspection
- **Documentation generation** with templated content

## Capabilities

### Template Expression Support
- **Input interpolation**: Access activity inputs using `{{inputs.*}}` syntax
- **Resource references**: Display resource properties via `{{inputs.resource.*}}`
- **Computed properties**: Show mirrored values using `{{inputs.*}}`
- **Multi-line content**: Support for formatted multi-line output

### Content Formatting
- **Plain text output**: Simple text content with variable substitution
- **Structured content**: YAML or JSON formatted output
- **Conditional content**: Template logic for conditional display
- **List formatting**: Display arrays and collections

### Expression Resolution
- **Runtime evaluation**: Expressions resolved during activity execution
- **Type preservation**: Maintain data types in output formatting
- **Error handling**: Graceful handling of undefined references

## Constraints

### Operational Limitations
- **Output-only**: No external system interaction or side effects
- **Text-based content**: Limited to textual output formats
- **No persistence**: Output is not stored or persisted
- **No state management**: Cannot maintain state between executions

### Expression Limitations
- **Read-only access**: Cannot modify input values or resources
- **Limited functions**: Basic template functions only
- **No complex logic**: Simple expression evaluation only

## Configuration Examples

### Basic Debug Activity
```yaml
apiVersion: core.clrslate.io
kind: Activity
metadata:
  name: debug-inputs
  title: Debug Activity Inputs
spec:
  inputs:
    type: object
    properties:
      appName:
        type: string
      environment:
        type: string
      cluster:
        $ref: "#/definitions/AKSCluster"
  handler:
    type: console
    properties:
      output: |
        === Debug Information ===
        Application: {{inputs.appName}}
        Environment: {{inputs.environment}}
        Target Cluster: {{inputs.cluster._name}}
        Cluster Region: {{inputs.cluster.location}}
```

### Configuration Verification
```yaml
apiVersion: core.clrslate.io
kind: Activity
metadata:
  name: verify-config
  title: Verify Deployment Configuration
spec:
  inputs:
    type: object
    properties:
      namespace:
        type: string
      replicas:
        type: integer
      config:
        type: object
  mirrored:
    totalReplicas:
      expression: "{{inputs.replicas * inputs.config.scaleFactor}}"
  handler:
    type: console
    properties:
      output: |
        Configuration Verification:
        - Namespace: {{inputs.namespace}}
        - Base Replicas: {{inputs.replicas}}
        - Scale Factor: {{inputs.config.scaleFactor}}
        - Total Replicas: {{inputs.totalReplicas}}
        
        Configuration is ready for deployment.
```

### Multi-Resource Status Report
```yaml
apiVersion: core.clrslate.io
kind: Activity
metadata:
  name: status-report
  title: Multi-Resource Status Report
spec:
  inputs:
    type: object
    properties:
      database:
        $ref: "#/definitions/AzureDatabase"
      storage:
        $ref: "#/definitions/StorageAccount"
      cluster:
        $ref: "#/definitions/AKSCluster"
  handler:
    type: console
    properties:
      output: |
        === Resource Status Report ===
        
        Database:
          Name: {{inputs.database._name}}
          Server: {{inputs.database.serverName}}
          Status: Available
        
        Storage:
          Name: {{inputs.storage._name}}
          Type: {{inputs.storage.accountType}}
          Status: Active
        
        Cluster:
          Name: {{inputs.cluster._name}}
          Version: {{inputs.cluster.kubernetesVersion}}
          Status: Running
        
        All resources are operational.
```

### Development Testing Activity
```yaml
apiVersion: core.clrslate.io
kind: Activity
metadata:
  name: test-expressions
  title: Test Expression Resolution
spec:
  inputs:
    type: object
    properties:
      testData:
        type: object
        properties:
          values:
            type: array
            items:
              type: string
          config:
            type: object
  mirrored:
    processedValues:
      expression: "{{inputs.testData.values | join(', ')}}"
  handler:
    type: console
    properties:
      output: |
        === Expression Testing ===
        Raw Values: {{inputs.testData.values}}
        Processed Values: {{inputs.processedValues}}
        Config Keys: {{inputs.testData.config | keys}}
        
        Expression resolution complete.
```

## Expression Patterns

### Common Expression Types

| Pattern | Purpose | Example | Output |
|---------|---------|---------|--------|
| `{{inputs.property}}` | Simple input access | `{{inputs.appName}}` | Direct value |
| `{{inputs.resource._name}}` | Resource name | `{{inputs.cluster._name}}` | Resource identifier |
| `{{inputs.resource.property}}` | Resource property | `{{inputs.cluster.location}}` | Property value |
| `{{inputs.computed}}` | Computed value | `{{inputs.fullName}}` | Calculated result |

### Advanced Formatting

```yaml
output: |
  {% if inputs.environment == "production" %}
  === PRODUCTION DEPLOYMENT ===
  Extra validation required.
  {% endif %}
  
  Application: {{inputs.appName}}
  Environment: {{inputs.environment}}
  
  {% for item in inputs.deploymentTargets %}
  - Target: {{item.name}} ({{item.region}})
  {% endfor %}
```

## Error Handling

### Common Error Patterns

| Error Type | Cause | Resolution |
|------------|-------|------------|
| `ExpressionError` | Invalid template syntax | Verify expression syntax and braces |
| `PropertyNotFound` | Referenced property doesn't exist | Check input schema and property names |
| `TypeMismatch` | Expression type incompatible | Ensure expression returns expected type |
| `TemplateError` | Template rendering failure | Validate template logic and conditionals |

### Validation Rules
- All template expressions must use valid `{{}}` syntax
- Referenced properties must exist in inputs, resources, or mirrored
- Template logic must be syntactically correct
- Output content must be serializable to text

## Performance Considerations

### Optimization Strategies
- **Simple expressions**: Keep expressions simple for faster evaluation
- **Minimal logic**: Avoid complex template logic in output
- **Property caching**: Cache frequently accessed properties
- **Content size**: Limit output content size for performance

### Best Practices
- Use Console handler for development and debugging only
- Prefer structured output over complex formatting
- Validate expressions during development
- Monitor output content size in production workflows

## Related Documentation

- [Execution Framework](execution-framework.md) - Core execution concepts
- [Tekton PipelineRef Handler](tekton-pipelineref.md) - Complex execution handler
- [Model Definition](model-definition.md) - Resource model specifications
- [Execution Examples](../execution-examples.md) - Complete activity examples

## Quick Reference

### Handler Type
```yaml
handler:
  type: console
```

### Required Properties
- `output`: Template content for display

### Expression Access Patterns
- Inputs: `{{inputs.propertyName}}`
- Resources: `{{inputs.resourceName._name}}`
- Mirrored: `{{inputs.computedProperty}}`

### Common Use Cases
- Debug input values
- Verify configurations
- Status reporting
- Development testing
- Documentation generation

### Validation Checklist
- [ ] Template syntax is valid
- [ ] All referenced properties exist
- [ ] Output content is reasonable size
- [ ] Expressions resolve correctly
- [ ] Template logic is syntactically correct
