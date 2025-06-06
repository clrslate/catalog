# Tekton PipelineRef Handler Examples

This document provides complete, working examples of the Tekton PipelineRef handler, grounded in the actual platform specifications.

## Complete PipelineRef Resource

Based on `invocationHandlers/tekton-pipelineref.md` and `models.cs`:

```yaml
apiVersion: tekton.clrslate.io
kind: PipelineRef
metadata:
  name: azure.pipelineRef.azure-script
  title: Azure Script
  description: A pipeline reference that allows you to run a script in an Azure environment
  tags:
    - tekton
    - azure
    - script
spec:
  schema:
    properties:
      credentials:
        type: object
        format: secret
        title: Azure Credentials
        description: Azure credentials for authentication
        specifications:
          type: azure.secret.clientCredentials
          mount: true
          mountParamName: azureCredentialsRef
      script:
        type: string
        title: Target Script
        description: Azure CLI script content to execute
      image:
        type: string
        title: Container Image
        description: Container image to use for script execution
        default: clrslatepublic.azurecr.io/tools/azure:latest
    required:
      - credentials
      - script
  mirrored:
    executionCommand:
      type: string
      value: |
        az config set core.only_show_errors=true --only-show-errors
        az login --service-principal -u $clientId -p $clientSecret --tenant $tenantId -o none
        az account set --subscription $subscriptionId -o none
        {{inputs.script}}
  pipeline:
    pipelineRef: azure.pipelines.azure-script
    params:
      script: '{{inputs.script}}'
      image: '{{inputs.image}}'
    secretMounts:
      azureCredentialsRef: "{{inputs.credentials._name}}"
    files:
      execution-script.sh: |
        #!/bin/bash
        set -e
        echo "Starting Azure script execution..."
        echo "Script content:"
        echo "{{inputs.script}}"
        echo "---"
        {{inputs.executionCommand}}
```

## Complete Activity with Tekton Handler

Based on `activity-specification.md` and the PipelineRef above:

```yaml
apiVersion: core.clrslate.io
kind: Activity
metadata:
  name: azure.activity.runScript
  title: Run Azure Script
  description: Execute Azure CLI script with credential mounting
  labels:
    category: Azure
  tags:
    - azure
    - script
    - tekton
    - infrastructure
spec:
  inputs:
    properties:
      credentials:
        type: object
        format: secret
        title: Azure Credentials
        description: Azure service principal credentials
        specifications:
          type: azure.secret.clientCredentials
      script:
        type: string
        title: Script Content
        description: Azure CLI script to execute
      cluster:
        type: object
        format: resource
        title: Target Cluster
        description: Kubernetes cluster for pipeline execution
        specifications:
          type: azure.model.aks
    required:
      - credentials
      - script
      - cluster
  resources:
    azureCreds:
      type: resource
      value: "{{inputs.credentials._name}}"
  handler:
    type: tekton.pipelineRef
    properties:
      pipeline: azure.pipelineRef.azure-script
      inputs:
        credentials: "{{inputs.credentials._name}}"
        script: "{{inputs.script}}"
        image: clrslatepublic.azurecr.io/tools/azure:latest
```

## Complete Corresponding Tekton Pipeline

Based on the pipeline reference in `invocationHandlers/tekton-pipelineref.md`:

```yaml
apiVersion: tekton.dev/v1
kind: Pipeline
metadata:
  name: azure.pipelines.azure-script
  labels:
    app.kubernetes.io/name: azure-script-pipeline
    clrslate.io/component: execution
spec:
  params:
  - name: azureCredentialsRef
    type: string
    description: Azure credentials secret reference
  - name: script
    type: string
    description: The script to run in the pipeline
  - name: image
    type: string
    description: The image to use for the script step
    default: clrslatepublic.azurecr.io/tools/azure:latest
  workspaces:
  - name: manifests
    description: Workspace for manifest files
  tasks:
  - name: script-task
    taskSpec:
      params:
      - name: azureCredentialsRef
        type: string
      - name: script
        type: string
      - name: image
        type: string
      workspaces:
      - name: manifests
      steps:
      - name: script-step
        image: $(params.image)
        script: |
          az config set core.only_show_errors=true --only-show-errors
          az login --service-principal -u $clientId -p $clientSecret --tenant $tenantId -o none
          az account set --subscription $subscriptionId -o none
          $(params.script)
        envFrom:
        - secretRef:
            name: $(params.azureCredentialsRef)
        volumeMounts:
        - name: manifests
          mountPath: /manifests
          readOnly: true
    params:
    - name: azureCredentialsRef
      value: $(params.azureCredentialsRef)
    - name: script
      value: $(params.script)
    - name: image
      value: $(params.image)
    workspaces:
    - name: manifests
      workspace: manifests
```

## Expression System Examples

### Resource Expression Access

When using resource expressions in Tekton activities:

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

When using computed properties in PipelineRef:

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
# This Tekton activity passes all validation rules
apiVersion: core.clrslate.io
kind: Activity
metadata:
  name: azure.activity.deployHelm  # ✅ Follows naming pattern
spec:
  inputs:
    properties:
      credentials:
        type: object
        format: secret
        specifications:
          type: azure.secret.clientCredentials
      chartName:
        type: string
    required:
      - credentials
      - chartName  # ✅ Required input defined
  handler:
    type: tekton.pipelineRef  # ✅ Supported handler type
    properties:
      pipeline: azure.pipelineRef.helm-deploy
      inputs:
        credentials: "{{inputs.credentials._name}}"  # ✅ Valid secret expression
        chartName: "{{inputs.chartName}}"  # ✅ Valid expression
```

### Activity Validation Failure Examples

```yaml
# Missing required input mapping for Tekton
spec:
  inputs:
    required: [credentials]
  handler:
    type: tekton.pipelineRef
    properties:
      inputs: {}  # ❌ Missing required input mapping

# Invalid secret expression
handler:
  type: tekton.pipelineRef
  properties:
    inputs:
      secret: "{{inputs.credentials}}"  # ❌ Should be: {{inputs.credentials._name}}

# Missing pipeline reference
handler:
  type: tekton.pipelineRef
  properties:
    inputs: {}  # ❌ Missing pipeline property
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
  type: tekton.pipelineRef
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

### Pipeline Optimization

```yaml
# Use workspaces efficiently
spec:
  workspaces:
  - name: manifests
    description: Shared workspace for manifest files

# Minimize task complexity
tasks:
- name: simple-task
  taskSpec:
    steps:
    - name: single-step  # Prefer single-step tasks for performance
      script: |
        # Combined operations reduce overhead
        az login && az deploy
```

## Advanced Patterns

### Multi-Stage Pipeline

```yaml
# PipelineRef for complex deployment workflow
apiVersion: tekton.clrslate.io
kind: PipelineRef
metadata:
  name: azure.pipelineRef.multi-stage-deploy
spec:
  schema:
    properties:
      environment:
        type: string
        enum: ["dev", "staging", "prod"]
      approvalRequired:
        type: boolean
        default: false
  pipeline:
    pipelineRef: azure.pipelines.multi-stage-deploy
    params:
      environment: '{{inputs.environment}}'
      approvalRequired: '{{inputs.approvalRequired}}'
    when:
    - input: "$(params.environment)"
      operator: in
      values: ["staging", "prod"]
      tasks: ["approval-task"]
```

### Conditional Execution

```yaml
# Activity with conditional pipeline execution
handler:
  type: tekton.pipelineRef
  properties:
    pipeline: azure.pipelineRef.conditional-deploy
    inputs:
      skipTests: "{{inputs.skipTests}}"
      environment: "{{inputs.environment}}"
    conditions:
      runTests:
        expression: "{{inputs.skipTests}} == false"
        tasks: ["test-task", "integration-task"]
```

These examples demonstrate the complete Tekton PipelineRef handler usage as documented in the platform specifications, including all features, constraints, and performance considerations.
