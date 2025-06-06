```prompt
---
mode: 'agent'
description: 'Create a new ClrSlate Tekton PipelineRef resource for complex pipeline execution'
---

# ClrSlate Tekton PipelineRef Creation Instructions

Your goal is to create a new ClrSlate Tekton PipelineRef resource that enables activities to execute complex workflows using Tekton pipelines with parameter mapping and secret handling capabilities.

## Information Gathering

First, collect the following required information from the user:

### Required PipelineRef Information
1. **PipelineRef Name**: Must follow qualified naming pattern `<domain>.pipelineRef.<name>` (e.g., "azure.pipelineRef.deployResourceGroup")
2. **Pipeline Purpose**: Clear description of what the pipeline accomplishes
3. **Target Tekton Pipeline**: Name of the Tekton pipeline that will be executed
4. **Required Parameters**: Input parameters needed for pipeline execution
5. **Secret Requirements**: Authentication credentials and mounting configuration
6. **File Templates**: Configuration files that need to be templated and injected

### Pipeline Execution Context
Ask these questions to understand the execution requirements:

**Infrastructure Requirements**:
- What cloud provider or infrastructure does this operate on?
- Does it require cluster access or specific RBAC permissions?
- What containerized tools or CLIs need to be available?

**Parameter Requirements**:
- What input parameters are required for execution?
- Are there optional parameters with default values?
- Do parameters need validation or transformation?

**Secret Requirements**:
- What authentication credentials are needed?
- How should secrets be mounted (file paths, environment variables)?
- Are there multiple secrets or credential types?

**File Template Requirements**:
- Are configuration files needed for the operation?
- What file formats are required (YAML, JSON, scripts)?
- Do files need dynamic content based on inputs?

## PipelineRef Structure Creation

### Basic PipelineRef Template
```yaml
apiVersion: tekton.clrslate.io
kind: PipelineRef
metadata:
  name: <qualified-name>              # domain.pipelineRef.name format
  title: <display-title>              # Human-readable title
  description: <pipeline-description> # Clear purpose description
  tags: [<operation-tags>]            # Operation categorization
spec:
  schema: <input-schema>              # Input validation and secret mounting
  mirrored: <computed-properties>     # Optional: Dynamic templating
  pipeline: <tekton-specification>    # Tekton pipeline execution config
```

### Schema Definition Requirements

Define comprehensive input schema with secret mounting:

```yaml
spec:
  schema:
    type: object
    properties:
      # Required resource references
      <resourceName>:
        type: string
        format: resource
        resourceType: <resource-type>
        description: <resource-purpose>
      
      # Configuration parameters
      <parameterName>:
        type: string
        description: <parameter-purpose>
        default: <default-value>       # Optional default
      
      # Secret references with mounting configuration
      credentials:
        type: string
        format: secret
        description: "Authentication credentials"
        mount: true                    # Enable secret mounting
        mountParamName: "credentials"  # Parameter name for pipeline
      
      # Additional secrets if needed
      <secretName>:
        type: string
        format: secret
        description: "<secret-purpose>"
        mount: true
        mountParamName: "<mount-param>"
    
    required: [<required-inputs>]      # List required inputs
```

### Mirrored Properties (Optional)

Use mirrored properties for computed values and dynamic templating:

```yaml
spec:
  mirrored:
    <computedProperty>:
      type: string
      description: "<computed-value-purpose>"
      # Mirrored properties enable dynamic value computation
```

### Pipeline Configuration

Define the Tekton pipeline execution specification:

```yaml
spec:
  pipeline:
    pipelineRef: <tekton-pipeline-name>    # Name of Tekton pipeline
    params:                                # Parameter mappings
      <pipeline-param>: "{{inputs.<input-name>}}"
      <resource-param>: "{{inputs.<resource-name>._name}}"
      <secret-param>: "{{inputs.<secret-name>._name}}"
    
    secretMounts:                          # Secret mounting configuration
      <mount-param-name>: "{{inputs.<secret-name>._name}}"
    
    files:                                 # File template mappings
      <file-path>: <file-template-content>
```

## Expression System Usage

### Parameter Mapping Patterns
Map activity inputs to pipeline parameters using expressions:

```yaml
params:
  # Direct input mapping
  namespace: "{{inputs.namespace}}"
  
  # Resource name mapping
  resourceGroupName: "{{inputs.resourceGroup._name}}"
  
  # Secret reference mapping
  azureCredentials: "{{inputs.credentials._name}}"
  
  # Complex property mapping
  location: "{{inputs.resourceGroup.spec.location}}"
  
  # Computed property mapping
  deploymentCommand: "{{inputs.computedCommand}}"
```

### Secret Mounting Configuration
Configure secrets for secure pipeline execution:

```yaml
secretMounts:
  # Mount secret as parameter reference
  credentials: "{{inputs.azureCredentials._name}}"
  
  # Mount additional secrets
  kubeconfig: "{{inputs.kubernetesCredentials._name}}"
```

### File Template Patterns
Define file templates for configuration injection:

```yaml
files:
  # YAML configuration template
  "/tmp/config.yaml": |
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: "{{inputs.configName}}"
      namespace: "{{inputs.namespace}}"
    data:
      value: "{{inputs.configValue}}"
  
  # Script template
  "/tmp/deploy.sh": |
    #!/bin/bash
    echo "Deploying to {{inputs.environment}}"
    kubectl apply -f /tmp/config.yaml -n "{{inputs.namespace}}"
  
  # JSON configuration template
  "/tmp/settings.json": |
    {
      "environment": "{{inputs.environment}}",
      "resourceGroup": "{{inputs.resourceGroup._name}}",
      "location": "{{inputs.resourceGroup.spec.location}}"
    }
```

## Naming Conventions

### Qualified PipelineRef Names
Follow the pattern: `<domain>.pipelineRef.<name>`

**Domain Examples**:
- `azure.pipelineRef.deployResourceGroup` - Azure resource group deployment
- `k8s.pipelineRef.createNamespace` - Kubernetes namespace creation
- `helm.pipelineRef.installChart` - Helm chart installation
- `github.pipelineRef.createRepository` - GitHub repository creation

### Operation Tags
Use descriptive tags to categorize pipeline operations:
- `deploy`, `create`, `delete`, `update` - CRUD operations
- `azure`, `kubernetes`, `helm`, `github` - Technology tags
- `infrastructure`, `application`, `configuration` - Scope tags

## Validation Requirements

Ensure created PipelineRef resources meet these validation rules:

### PipelineRef Validation
- [ ] **PIP-001**: PipelineRef name follows qualified naming pattern `<domain>.pipelineRef.<name>`
- [ ] **PIP-002**: Secret mount configuration is complete (mount: true + mountParamName)
- [ ] **PIP-003**: Pipeline reference exists in Tekton cluster
- [ ] **PIP-004**: Secret mount parameters match schema definitions

### Schema Validation
- [ ] All resource references specify `format: resource` and `resourceType`
- [ ] Secret inputs specify `format: secret` with complete mounting configuration
- [ ] Required inputs are listed in `required` array
- [ ] Parameter descriptions are clear and descriptive

### Pipeline Configuration Validation
- [ ] Parameter mappings use correct expression syntax
- [ ] Secret mounting references valid schema properties
- [ ] File templates use proper expression substitution
- [ ] All mapped parameters are defined in schema

## Integration with Activities

PipelineRef resources are consumed by activities using the `tekton.pipelineRef` handler:

```yaml
# Activity that uses this PipelineRef
handler:
  type: tekton.pipelineRef
  properties:
    pipeline: <pipelineref-resource-name>  # Reference to this PipelineRef
    inputs: {}                             # Parameter mappings handled by PipelineRef
```

Ensure that:
1. **Schema Alignment**: Activity inputs match PipelineRef schema requirements
2. **Secret Consistency**: Secret mounting is configured properly in both
3. **Parameter Coverage**: All required PipelineRef inputs are provided by activity

## Example Templates

### Basic Infrastructure PipelineRef
```yaml
apiVersion: tekton.clrslate.io
kind: PipelineRef
metadata:
  name: "azure.pipelineRef.createResourceGroup"
  title: "Create Azure Resource Group"
  description: "Creates an Azure resource group using Tekton pipeline"
  tags: ["azure", "create", "infrastructure"]
spec:
  schema:
    type: object
    properties:
      resourceGroup:
        type: string
        format: resource
        resourceType: "azure.resourceGroup"
        description: "Azure resource group to create"
      credentials:
        type: string
        format: secret
        description: "Azure service principal credentials"
        mount: true
        mountParamName: "azureCredentials"
    required: ["resourceGroup", "credentials"]
  
  pipeline:
    pipelineRef: "azure-create-resource-group"
    params:
      resourceGroupName: "{{inputs.resourceGroup._name}}"
      location: "{{inputs.resourceGroup.spec.location}}"
    secretMounts:
      azureCredentials: "{{inputs.credentials._name}}"
```

### Complex Deployment PipelineRef
```yaml
apiVersion: tekton.clrslate.io
kind: PipelineRef
metadata:
  name: "helm.pipelineRef.deployChart"
  title: "Deploy Helm Chart"
  description: "Deploys a Helm chart to Kubernetes cluster"
  tags: ["helm", "deploy", "kubernetes"]
spec:
  schema:
    type: object
    properties:
      chart:
        type: string
        format: resource
        resourceType: "helm.chart"
        description: "Helm chart to deploy"
      cluster:
        type: string
        format: resource
        resourceType: "k8s.cluster"
        description: "Target Kubernetes cluster"
      namespace:
        type: string
        description: "Target namespace"
        default: "default"
      kubeconfig:
        type: string
        format: secret
        description: "Kubernetes cluster credentials"
        mount: true
        mountParamName: "kubeconfig"
    required: ["chart", "cluster", "kubeconfig"]
  
  pipeline:
    pipelineRef: "helm-deploy-chart"
    params:
      chartName: "{{inputs.chart._name}}"
      chartVersion: "{{inputs.chart.spec.version}}"
      namespace: "{{inputs.namespace}}"
      releaseName: "{{inputs.chart._name}}-{{inputs.namespace}}"
    secretMounts:
      kubeconfig: "{{inputs.kubeconfig._name}}"
    files:
      "/tmp/values.yaml": |
        # Generated values for {{inputs.chart._name}}
        namespace: "{{inputs.namespace}}"
        cluster: "{{inputs.cluster._name}}"
```

## Troubleshooting Common Issues

### Parameter Mapping Errors
- **Issue**: `ValidationError` - Invalid parameter mapping
- **Solution**: Verify expression syntax and ensure referenced inputs exist in schema

### Secret Mounting Errors
- **Issue**: `SecretMountError` - Secret mounting configuration invalid
- **Solution**: Ensure `mount: true` and valid `mountParamName` in schema

### Pipeline Execution Errors
- **Issue**: `ExecutionTimeout` - Pipeline execution exceeded timeout
- **Solution**: Optimize pipeline or increase timeout configuration

### File Template Errors
- **Issue**: Template substitution failures
- **Solution**: Verify expression syntax and ensure all referenced properties exist

## Example Workflow

1. **Gather Requirements**: Collect pipeline purpose, parameters, and secret needs
2. **Define Schema**: Create comprehensive input validation with secret mounting
3. **Configure Pipeline**: Set up Tekton pipeline execution specification
4. **Map Parameters**: Create proper expression mappings for all inputs
5. **Template Files**: Define any configuration file templates needed
6. **Validate**: Ensure all validation rules are met
7. **Test Integration**: Verify the PipelineRef works with corresponding activities

Ask for the pipeline requirements if not provided, then follow this structured approach to create a complete, standards-compliant ClrSlate Tekton PipelineRef resource.

This PipelineRef can then be referenced by activities using the create-package-activity prompt with the `tekton.pipelineRef` handler type.
```
