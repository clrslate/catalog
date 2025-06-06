---
type: reference
category: execution
subcategory: handlers
complexity: intermediate
prerequisites: [execution-framework, model-definition]
outputs: [tekton-activities, pipeline-configurations]
lastUpdated: 2025-05-28
version: 1.0.0
---

# Tekton PipelineRef Handler

The Tekton PipelineRef handler enables activities to execute complex workflows using Tekton pipelines with parameter mapping and secret handling capabilities.

## Overview

The Tekton PipelineRef handler provides a bridge between ClrSlate activities and Tekton pipeline infrastructure, allowing for containerized execution environments with full parameter mapping, secret mounting, and file templating support.

## Handler Configuration

### Basic Pattern
```yaml
handler:
  type: tekton.pipelineRef
  properties:
    pipeline: <pipelineref-resource-name>
    inputs: <parameter-mappings>
```

### Complete Configuration
```yaml
handler:
  type: tekton.pipelineRef
  properties:
    pipeline: deploy-helm-chart
    inputs:
      namespace: "{{inputs.namespace}}"
      chartName: "{{inputs.chartName}}"
      cluster: "{{inputs.cluster._name}}"
      values: "{{inputs.helmValues}}"
```

## PipelineRef Resource Model

The Tekton PipelineRef is a specialized wrapper model that defines how to invoke Tekton pipelines:

```yaml
apiVersion: tekton.clrslate.io
kind: PipelineRef
metadata:
  name: <pipeline-ref-name>
  title: <display-title>
  description: <description>
  tags: [<operation-tags>]
spec:
  schema: <input-schema>
  mirrored: <computed-properties>  # optional
  pipeline: <tekton-pipeline-specification>
```

### PipelineRef Specification Fields

| Field | Purpose | Required | Description |
|-------|---------|----------|-------------|
| `schema` | Input validation | Yes | Input schema with parameter validation and secret mounting configuration |
| `mirrored` | Computed properties | No | Dynamic templating and transformation logic |
| `pipeline` | Execution spec | Yes | Tekton pipeline execution specification with parameter mappings |

### Tekton Pipeline Specification

```yaml
pipeline:
  pipelineRef: <tekton-pipeline-name>
  params: <parameter-mappings>
  secretMounts: <secret-mount-references>
  files: <file-template-mappings>
```

**Component Details:**
- **`pipelineRef`**: Name of the Tekton pipeline to execute
- **`params`**: Parameter mappings from activity inputs to pipeline parameters
- **`secretMounts`**: Configuration for mounting secrets into pipeline execution
- **`files`**: File template mappings for configuration injection

## When to Use

The Tekton PipelineRef handler is optimal for:

- **Complex execution logic** requiring containerized environments
- **Multi-step operations** with dependencies and conditional logic
- **Operations requiring Azure/cloud credentials** with secure secret handling
- **Infrastructure provisioning** or deployment tasks
- **CI/CD workflows** requiring pipeline orchestration

## Capabilities

### Parameter Mapping
- **Expression support**: Use `{{}}` syntax for dynamic parameter resolution
- **Input mapping**: Direct access to activity inputs via `{{inputs.*}}`
- **Resource references**: Access to resource properties via `{{inputs.resource._name}}`
- **Computed properties**: Access to mirrored values via `{{inputs.*}}`

### Secret Handling
- **Automatic mounting**: Secrets specified in schema are automatically mounted
- **Secure execution**: Credentials are injected at runtime without exposure
- **Mount path configuration**: Flexible secret mounting paths for different tools

### File Templating
- **Configuration injection**: Template files with dynamic content
- **Multi-format support**: Support for YAML, JSON, and text file templates
- **Runtime resolution**: Templates resolved during pipeline execution

### Pipeline Validation
- **Parameter validation**: Input validation before pipeline execution
- **Schema enforcement**: Strict adherence to PipelineRef schema definitions
- **Error handling**: Comprehensive error reporting for validation failures

## Constraints

### Infrastructure Requirements
- **Tekton cluster**: Requires active Tekton pipeline infrastructure
- **RBAC permissions**: Appropriate service account permissions for pipeline execution
- **Resource availability**: Sufficient cluster resources for pipeline workloads

### Configuration Limitations
- **PipelineRef dependency**: Limited to predefined PipelineRef resource definitions
- **Secret specifications**: Secrets must include mount configuration in schema
- **Pipeline compatibility**: Target pipelines must accept mapped parameters

### Execution Constraints
- **Async execution**: Pipeline execution is asynchronous with status tracking
- **Resource limits**: Subject to cluster resource quotas and limits
- **Timeout handling**: Pipeline execution timeouts must be managed

## Configuration Examples

### Basic Deployment Activity
```yaml
apiVersion: core.clrslate.io
kind: Activity
metadata:
  name: deploy-application
  title: Deploy Application via Tekton
spec:
  inputs:
    type: object
    properties:
      namespace:
        type: string
      appName:
        type: string
      cluster:
        $ref: "#/definitions/AKSCluster"
  handler:
    type: tekton.pipelineRef
    properties:
      pipeline: deployment-pipeline
      inputs:
        namespace: "{{inputs.namespace}}"
        applicationName: "{{inputs.appName}}"
        targetCluster: "{{inputs.cluster._name}}"
```

### Advanced Configuration with Secrets
```yaml
apiVersion: tekton.clrslate.io
kind: PipelineRef
metadata:
  name: helm-deployment
  title: Helm Chart Deployment Pipeline
spec:
  schema:
    type: object
    properties:
      namespace:
        type: string
      chartName:
        type: string
      cluster:
        $ref: "#/definitions/AKSCluster"
      credentials:
        $ref: "#/definitions/AzureCredentials"
        secretMount:
          path: "/etc/azure"
          mode: "0600"
  pipeline:
    pipelineRef: helm-deploy-pipeline
    params:
      - name: namespace
        value: "{{inputs.namespace}}"
      - name: chart-name
        value: "{{inputs.chartName}}"
      - name: kubeconfig
        value: "{{inputs.cluster.kubeconfig}}"
    secretMounts:
      - secret: "{{inputs.credentials._name}}"
        mountPath: "/etc/azure"
    files:
      - name: values.yaml
        template: |
          image:
            repository: "{{inputs.chartName}}"
            tag: "latest"
          namespace: "{{inputs.namespace}}"
```

## Error Handling

### Common Error Patterns

| Error Type | Cause | Resolution |
|------------|-------|------------|
| `ValidationError` | Invalid parameter mapping | Verify input schema and parameter names |
| `PipelineNotFound` | Referenced pipeline doesn't exist | Ensure pipeline is deployed and accessible |
| `SecretMountError` | Secret mounting configuration invalid | Verify secret exists and mount path is valid |
| `ExecutionTimeout` | Pipeline execution exceeded timeout | Increase timeout or optimize pipeline |

### Validation Rules
- All referenced PipelineRef resources must exist and be accessible
- Parameter mappings must resolve to valid values
- Secret specifications must include valid mount configurations
- Template expressions must use supported syntax and reference valid properties

## Performance Considerations

### Optimization Strategies
- **Pipeline efficiency**: Design pipelines for optimal resource utilization
- **Parameter caching**: Cache resolved parameters for repeated executions
- **Secret management**: Minimize secret mounting overhead
- **Resource quotas**: Configure appropriate resource limits and requests

### Monitoring
- **Pipeline status**: Monitor pipeline execution status and duration
- **Resource usage**: Track cluster resource consumption
- **Error rates**: Monitor validation and execution error rates
- **Performance metrics**: Collect execution time and success rate metrics

## Related Documentation

- [Execution Framework](execution-framework.md) - Core execution concepts
- [Model Definition](model-definition.md) - Resource model specifications
- [Secret Definition](secret-definition.md) - Secret handling patterns
- [Console Handler](console.md) - Alternative handler for simple operations
- [Complete Tekton PipelineRef Handler Examples](examples.md) - Comprehensive examples covering various use cases and configurations

## Quick Reference

### Handler Type
```yaml
handler:
  type: tekton.pipelineRef
```

### Required Properties
- `pipeline`: PipelineRef resource name
- `inputs`: Parameter mapping configuration

### Expression Access Patterns
- Inputs: `{{inputs.propertyName}}`
- Resources: `{{inputs.resourceName._name}}`
- Mirrored: `{{inputs.computedProperty}}`

### Validation Checklist
- [ ] PipelineRef resource exists and is accessible
- [ ] All parameter mappings resolve correctly
- [ ] Secret mount configurations are valid
- [ ] Pipeline accepts all mapped parameters
- [ ] Execution permissions are configured
