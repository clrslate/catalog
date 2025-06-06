---
mode: 'agent'
description: 'Create a new ClrSlate activity with proper execution handler configuration'
---

# ClrSlate Activity Creation Instructions

Your goal is to create a new ClrSlate activity following the execution framework standards and handler patterns.

## Information Gathering

First, collect the following required information from the user:

### Required Activity Information
1. **Activity Name**: Must follow qualified naming pattern `<domain>.<category>.<name>` (e.g., "azure.deployment.createResourceGroup")
2. **Activity Purpose**: Clear description of what the activity accomplishes
3. **Operation Type**: Type of operation (create, delete, deploy, configure, etc.)
4. **Target Resources**: What resources or systems the activity operates on
5. **Execution Complexity**: Simple output/debugging vs complex operations requiring infrastructure

### Handler Selection Criteria
Ask these questions to determine the appropriate handler:

**For Console Handler** (simple operations):
- Is this primarily for output, debugging, or status display?
- Does it require no external system interaction?
- Is it for development/testing purposes?

**For Tekton PipelineRef Handler** (complex operations):
- Does it require cluster access or cloud credentials?
- Is it a multi-step operation with dependencies?
- Does it provision infrastructure or deploy applications?
- Does it need containerized execution environment?

**For Helm Chart Installation** (special case):
- Is this specifically for installing/upgrading a Helm chart?
- Does it target a specific chart (MongoDB, Grafana, Istio, etc.)?
- Should it use the standardized `helm.pipelineRef.helm-install` pipeline?

*If creating a Helm chart installation activity, follow the specialized workflow in the "Special Case: Helm Chart Installation Activities" section below.*

## Activity Structure Creation

### Basic Activity Template
```yaml
apiVersion: core.clrslate.io
kind: Activity
metadata:
  name: <qualified-name>              # domain.category.name format
  title: <display-title>              # Human-readable title
  description: <activity-description> # Clear purpose description
  tags: [<operation-tags>]            # Operation categorization
spec:
  inputs: <input-schema>              # Input validation schema
  handler: <execution-handler>        # Handler configuration
```

### Input Schema Requirements

Define comprehensive input validation:

```yaml
spec:
  inputs:
    type: object
    properties:
      # Required resource references (MUST use this object pattern)
      <resourceName>:
        type: object
        format: resource
        title: <Resource Title>
        description: <Resource Purpose>
        specifications:
          type: <resource-type>  # e.g., azure.model.aks
      # All resource definitions MUST include title and description fields for clarity and documentation.
      
      # Configuration parameters
      <parameterName>:
        type: string
        description: <parameter-purpose>
        default: <default-value>       # Optional default
      
      # Secret references (for Tekton handlers)
      credentials:
        type: object
        format: secret
        title: <Secret Title>
        description: <Secret Purpose>
        specifications:
          type: <secret-type>  # e.g., azure.secret.clientCredentials
          mount: true
          mountParamName: <mount-param>  # Parameter name for mounting
    required: [<required-inputs>]      # List required inputs
```

## Handler Configuration

### Console Handler Configuration
Use for simple output, debugging, and status display:

```yaml
handler:
  type: console
  properties:
    message: <output-message>          # Static or templated message
    inputs: <input-mappings>           # Access to activity inputs
```

**Console Handler Patterns**:
- **Static output**: Simple status messages
- **Dynamic output**: Use `{{inputs.propertyName}}` expressions
- **Resource display**: Show resource information with `{{inputs.resourceName._name}}`

### Tekton PipelineRef Handler Configuration
Use for complex operations requiring infrastructure:

```yaml
handler:
  type: tekton.pipelineRef
  properties:
    pipeline: <pipelineref-resource-name>  # Reference to PipelineRef resource
    inputs: <parameter-mappings>           # Map inputs to pipeline parameters
```

**PipelineRef Handler Requirements**:
- Requires corresponding PipelineRef resource (create separately or link to existing)
- Parameter mappings must match PipelineRef schema
- Secret inputs require proper mounting configuration

## Naming Conventions

### Qualified Activity Names
Follow the pattern: `<domain>.<category>.<name>`

**Domain Examples**:
- `azure` - Azure cloud operations
- `k8s` - Kubernetes operations  
- `helm` - Helm chart operations
- `github` - GitHub operations

**Category Examples**:
- `deployment` - Deployment operations
- `configuration` - Configuration management
- `monitoring` - Monitoring and observability
- `security` - Security and access management

**Name Examples**:
- `createResourceGroup` - Create Azure resource group
- `deployChart` - Deploy Helm chart
- `configureIngress` - Configure ingress rules

### Tags for Categorization
Use descriptive tags to categorize operations:
- `create`, `delete`, `update` - CRUD operations
- `deploy`, `configure`, `install` - Deployment operations
- `azure`, `kubernetes`, `helm` - Technology tags

## Expression System Usage

### Resource References
Access resource properties using expression syntax:

```yaml
# Resource name access
"{{inputs.cluster._name}}"

# Resource specification access  
"{{inputs.resourceGroup.spec.location}}"

# Resource metadata access
"{{inputs.resource._metadata.name}}"
```

### Input Parameters
Access activity inputs directly:

```yaml
# Simple parameter access
"{{inputs.namespace}}"

# Complex parameter access
"{{inputs.configuration.replicas}}"
```

### Computed Properties
Access mirrored/computed values:

```yaml
# Computed from PipelineRef mirrored properties
"{{inputs.computedCommand}}"
```

## Special Case: Helm Chart Installation Activities

When creating activities specifically for Helm chart installation/upgrade operations, follow this specialized workflow:

### Helm Chart Activity Detection
Identify when to use the Helm chart installation pattern:
- Activity purpose includes "install", "deploy", "upgrade" with Helm charts
- Target is a specific Helm chart (e.g., MongoDB, Grafana, Istio components)
- Operation requires Kubernetes cluster access and Helm execution

### Two-Step Process for Helm Chart Activities

#### Step 1: Create or Identify HelmChart Record
Before creating the activity, ensure the HelmChart Record exists:

**Use the create-helmChart prompt** to create the HelmChart Record if it doesn't exist:
1. Gather chart information (name, version, repository)
2. Create Record following `<domain>.records.helmChart.<instance-name>` pattern
3. Place in appropriate package (e.g., `istio/resources/helmCharts/`, `mongodb/resources/helmCharts/`)

**Example HelmChart Records**:
```yaml
# istio.records.helmChart.istio-base
apiVersion: records.clrslate.io
kind: helm.model.helmChart
metadata:
  name: istio.records.helmChart.istio-base
spec:
  name: istio/base
  version: "1.20.0"
  repository: https://istio-release.storage.googleapis.com/charts
  namespace: istio-system

# clrslatePlatform.records.helmChart.clrslate-mongo  
apiVersion: records.clrslate.io
kind: helm.model.helmChart
metadata:
  name: clrslatePlatform.records.helmChart.clrslate-mongo
spec:
  name: oci://clrslatepublic.azurecr.io/helm/mongodb
  version: "1.0.1"
  namespace: mongodb
```

#### Step 2: Create Helm Installation Activity
Create the activity using the standardized Helm installation pattern:

**Required Inputs for Helm Activities**:
```yaml
spec:
  inputs:
    properties:
      cluster:
        type: object
        format: resource
        title: Cluster
        description: Target Kubernetes Cluster
        specifications:
          type: azure.model.aks
      namespace:
        type: string
        description: The namespace of the Helm release
        # Optional: make this optional if chart has default namespace
    required:
      - cluster
      # Include namespace in required only if chart doesn't specify default
```

**Standardized Handler Configuration**:
```yaml
handler:
  type: tekton.pipelineRef
  properties:
    pipeline: helm.pipelineRef.helm-install  # Use existing helm-install PipelineRef
    inputs: 
      cluster: "{{inputs.cluster._name}}"
      helmChart: "<domain>.records.helmChart.<chart-instance>"  # Reference to HelmChart Record
      releaseName: "<helm-release-name>"     # Logical name for the release
      namespace: "{{inputs.namespace}}"      # Dynamic namespace or chart default
```

### Helm Chart Activity Template
```yaml
apiVersion: core.clrslate.io
kind: Activity
metadata:
  name: <domain>.activity.deploy<ChartName>
  title: Deploy <Chart Display Name>
  description: Deploy <Chart Display Name> Helm Chart
  labels:
    category: <Domain Category>
  tags:
    - create
    - helm
    - deploy
spec:
  inputs:
    properties:
      cluster:
        type: object
        format: resource
        title: Cluster
        description: Target Kubernetes Cluster
        specifications:
          type: azure.model.aks
      namespace:
        type: string
        description: The namespace of the Helm release
        # Make optional if chart specifies default namespace
    required:
      - cluster
      # Only include namespace if required (chart has no default)
  handler:
    type: tekton.pipelineRef
    properties:
      pipeline: helm.pipelineRef.helm-install
      inputs: 
        cluster: "{{inputs.cluster._name}}"
        helmChart: "<domain>.records.helmChart.<instance-name>"
        releaseName: "<logical-release-name>"
        namespace: "{{inputs.namespace}}"
        # Optional: Add createNamespace if needed
        # createNamespace: true
```

### Helm Activity Examples

**Istio Base Deployment**:
```yaml
apiVersion: core.clrslate.io
kind: Activity
metadata:
  name: istio.activity.deployIstioBase
  title: Deploy Istio Base
  description: Deploy Istio Base Helm Chart
  labels:
    category: Istio
  tags:
    - create
spec:
  inputs:
    properties:
      cluster:
        type: object
        format: resource
        title: Cluster
        description: Target Kubernetes Cluster
        specifications:
          type: azure.model.aks
    required:
      - cluster
  handler:
    type: tekton.pipelineRef
    properties:
      pipeline: helm.pipelineRef.helm-install
      inputs: 
        cluster: "{{inputs.cluster._name}}"
        helmChart: "istio.records.helmChart.istio-base"
        releaseName: istio-base
        namespace: "{{inputs.namespace}}"
```

**MongoDB Deployment with Required Namespace**:
```yaml
apiVersion: core.clrslate.io
kind: Activity
metadata:
  name: clrslatePlatform.activity.deployClrSlateMongo
  title: Deploy ClrSlate Mongo
  description: Deploy ClrSlate Mongo
  labels:
    category: ClrSlate Platform
  tags:
    - create
spec:
  inputs:
    properties:
      cluster:
        type: object
        format: resource
        title: Cluster
        description: Target Kubernetes Cluster
        specifications:
          type: azure.model.aks
      namespace:
        type: string
        description: The namespace of the Helm release
    required:
      - cluster
  handler:
    type: tekton.pipelineRef
    properties:
      pipeline: helm.pipelineRef.helm-install
      inputs: 
        cluster: "{{inputs.cluster._name}}"
        helmChart: "clrslatePlatform.records.helmChart.clrslate-mongo"
        releaseName: mongo
        namespace: "{{inputs.namespace}}"
```

### Naming Conventions for Helm Activities

**Activity Naming Pattern**:
- `<domain>.activity.deploy<ChartName>` (e.g., `istio.activity.deployIstioBase`)
- Use PascalCase for chart name portion
- Prefix with "deploy" for installation activities

**HelmChart Record Reference**:
- Always reference the full Record name: `<domain>.records.helmChart.<instance-name>`
- Ensure Record exists before creating activity

**Release Naming**:
- Use logical, consistent release names
- Consider environment-specific patterns if needed
- Keep names DNS-compliant (lowercase, hyphens)

### Validation for Helm Chart Activities

Additional validation rules for Helm chart activities:

- [ ] **HELM-001**: HelmChart Record exists and is referenced correctly
- [ ] **HELM-002**: Uses `helm.pipelineRef.helm-install` as the pipeline
- [ ] **HELM-003**: Release name follows DNS naming conventions
- [ ] **HELM-004**: Cluster input uses `azure.model.aks` specification
- [ ] **HELM-005**: Namespace handling is appropriate (required vs optional)
- [ ] **HELM-006**: Activity name follows `<domain>.activity.deploy<ChartName>` pattern

## Validation Requirements

Ensure created activities meet these validation rules:

### Activity Validation
- [ ] **ACT-001**: Activity name follows qualified naming pattern `<domain>.<category>.<name>`
- [ ] **ACT-002**: Handler type is supported (`tekton.pipelineRef` or `console`)
- [ ] **ACT-003**: PipelineRef resource exists when using `tekton.pipelineRef` handler
- [ ] **ACT-004**: Required inputs are provided in handler parameter mappings
- [ ] **ACT-005**: Secret inputs use proper expression format with `._name` suffix

### Input Schema Validation
- [ ] All resource references use `type: object`, `format: resource`, and include a `specifications` section with the correct `type` (e.g., `azure.model.aks`)
- [ ] All secret references use `type: object`, `format: secret`, and include a `specifications` section with the correct `type` (e.g., `azure.secret.clientCredentials`), `mount: true`, and `mountParamName`.
- [ ] Required inputs are listed in `required` array
- [ ] Input descriptions are clear and descriptive

### Handler Configuration Validation
- [ ] Console handlers have appropriate message templates
- [ ] Tekton handlers reference valid PipelineRef resources
- [ ] Parameter mappings use correct expression syntax
- [ ] Secret mounting is properly configured for Tekton handlers, and secret parameter mappings use the `{{inputs.secretName._name}}` pattern

## Integration with PipelineRef

When creating activities that use `tekton.pipelineRef` handler:

1. **Create PipelineRef First**: Use the create-tekton-pipelineref prompt to create the PipelineRef resource
2. **Match Schemas**: Ensure activity inputs align with PipelineRef schema
3. **Parameter Mapping**: Map activity inputs to PipelineRef parameters correctly
4. **Secret Handling**: Configure secret mounting consistently between activity and PipelineRef

## Example Workflow

### Standard Activity Creation
1. **Gather Information**: Collect activity requirements and determine handler type
2. **Create Structure**: Generate activity YAML with proper structure
3. **Define Inputs**: Create comprehensive input schema with validation
4. **Configure Handler**: Set up appropriate handler configuration
5. **Validate**: Ensure all validation rules are met
6. **Link Resources**: If using Tekton handler, ensure PipelineRef resource exists

### Helm Chart Activity Creation
1. **Detect Helm Pattern**: Identify if this is a Helm chart installation activity
2. **Create/Identify HelmChart Record**: Use create-helmChart prompt if Record doesn't exist
3. **Follow Helm Template**: Use the standardized Helm activity template
4. **Reference HelmChart**: Ensure correct Record reference in `helmChart` parameter
5. **Configure Release**: Set appropriate release name and namespace handling
6. **Validate Helm Rules**: Check Helm-specific validation requirements

## Templates

### Console Activity Template
```yaml
apiVersion: core.clrslate.io
kind: Activity
metadata:
  name: "<domain>.<category>.<name>"
  title: "<Display Title>"
  description: "<Activity purpose and functionality>"
  tags: ["<operation>", "<technology>"]
spec:
  inputs:
    type: object
    properties:
      message:
        type: string
        description: "Message to display"
        default: "Operation completed successfully"
    required: []
  handler:
    type: console
    properties:
      message: "{{inputs.message}}"
```

### Tekton Activity Template
```yaml
apiVersion: core.clrslate.io
kind: Activity
metadata:
  name: "<domain>.<category>.<name>"
  title: "<Display Title>"
  description: "<Activity purpose and functionality>"
  tags: ["<operation>", "<technology>"]
spec:
  inputs:
    type: object
    properties:
      credentials:
        type: string
        format: secret
        description: "Authentication credentials"
        mount: true
        mountParamName: "credentials"
      # Additional inputs as needed
    required: ["credentials"]
  handler:
    type: tekton.pipelineRef
    properties:
      pipeline: "<pipelineref-resource-name>"
      inputs: {} # Parameter mappings
```

Ask for the activity requirements if not provided, determine the appropriate handler type based on complexity and requirements, then follow this structured approach to create a complete, standards-compliant ClrSlate activity.

**For Helm chart installation activities**: Follow the specialized two-step workflow using the create-helmChart prompt first, then the standardized Helm activity template.

**For complex operations requiring Tekton execution**: Ensure the corresponding PipelineRef resource is created using the create-tekton-pipelineref prompt.