---
type: examples
category: execution
subcategory: handlers
handler: console
complexity: beginner
prerequisites: [console-handler]
outputs: [console-activities, debug-workflows]
lastUpdated: 2025-05-28
version: 1.0.0
---

# Console Handler Examples

Complete working examples for the Console handler, grounded in actual ClrSlate platform specifications.

## Basic Debug Activity

Based on `activity-specification.md` and `invocationHandlers/console.md`:

```yaml
apiVersion: core.clrslate.io
kind: Activity
metadata:
  name: debug.activity.showConfig
  title: Show Configuration
  description: Display configuration values for debugging
  labels:
    category: Debug
  tags:
    - debug
    - console
    - validation
spec:
  inputs:
    properties:
      environment:
        type: string
        title: Environment
        description: Target environment name
        enum: ["dev", "staging", "prod"]
      namespace:
        type: string
        title: Namespace
        description: Target namespace
        default: default
      cluster:
        type: object
        format: resource
        title: Target Cluster
        description: Kubernetes cluster for validation
        specifications:
          type: azure.model.aks
    required:
      - environment
      - cluster
  handler:
    type: console
    properties:
      output: |
        🔍 Configuration Display
        =======================
        Environment: {{inputs.environment}}
        Namespace: {{inputs.namespace}}
        Cluster: {{inputs.cluster._name}} ({{inputs.cluster.resourceGroup.name}})
        
        ✅ Configuration validated and ready for use.
```

## Multi-Resource Status Report

```yaml
apiVersion: core.clrslate.io
kind: Activity
metadata:
  name: debug.activity.resourceStatus
  title: Resource Status Report
  description: Display status of multiple resources
  labels:
    category: Debug
  tags:
    - debug
    - console
    - status
    - resources
spec:
  inputs:
    properties:
      database:
        type: object
        format: resource
        title: Database Resource
        description: Azure database resource
        specifications:
          type: azure.model.database
      storage:
        type: object
        format: resource
        title: Storage Account
        description: Azure storage account
        specifications:
          type: azure.model.storage
      cluster:
        type: object
        format: resource
        title: AKS Cluster
        description: Kubernetes cluster resource
        specifications:
          type: azure.model.aks
    required:
      - database
      - storage
      - cluster
  handler:
    type: console
    properties:
      output: |
        === Resource Status Report ===
        Generated: {{timestamp}}
        
        Database:
          Name: {{inputs.database._name}}
          Server: {{inputs.database.serverName}}
          Resource Group: {{inputs.database.resourceGroup.name}}
          Status: Available
        
        Storage:
          Name: {{inputs.storage._name}}
          Type: {{inputs.storage.accountType}}
          Resource Group: {{inputs.storage.resourceGroup.name}}
          Status: Active
        
        Cluster:
          Name: {{inputs.cluster._name}}
          Version: {{inputs.cluster.kubernetesVersion}}
          Resource Group: {{inputs.cluster.resourceGroup.name}}
          Status: Running
        
        All resources are operational.
```

## Input Validation Display

```yaml
apiVersion: core.clrslate.io
kind: Activity
metadata:
  name: debug.activity.validateInputs
  title: Validate Activity Inputs
  description: Display and validate activity input values
  labels:
    category: Debug
  tags:
    - debug
    - console
    - validation
    - development
spec:
  inputs:
    properties:
      appName:
        type: string
        title: Application Name
        description: Name of the application to deploy
        pattern: "^[a-z][a-z0-9-]*[a-z0-9]$"
      replicas:
        type: integer
        title: Replica Count
        description: Number of application replicas
        minimum: 1
        maximum: 10
        default: 2
      config:
        type: object
        title: Configuration
        description: Application configuration object
        properties:
          scaleFactor:
            type: number
            minimum: 0.1
            maximum: 5.0
            default: 1.0
          enableMetrics:
            type: boolean
            default: true
    required:
      - appName
  mirrored:
    totalReplicas:
      type: integer
      value: "{{inputs.replicas * inputs.config.scaleFactor}}"
    validationSummary:
      type: string
      value: |
        App: {{inputs.appName}} | Replicas: {{inputs.replicas}} | Scale: {{inputs.config.scaleFactor}} | Total: {{inputs.totalReplicas}}
  handler:
    type: console
    properties:
      output: |
        === Input Validation Report ===
        
        📋 Basic Inputs:
        - Application Name: {{inputs.appName}}
        - Base Replicas: {{inputs.replicas}}
        - Metrics Enabled: {{inputs.config.enableMetrics}}
        
        ⚙️ Configuration:
        - Scale Factor: {{inputs.config.scaleFactor}}
        - Calculated Total Replicas: {{inputs.totalReplicas}}
        
        📊 Summary:
        {{inputs.validationSummary}}
        
        ✅ All inputs validated successfully.
```

## Development Testing Activity

```yaml
apiVersion: core.clrslate.io
kind: Activity
metadata:
  name: debug.activity.testExpressions
  title: Test Expression Resolution
  description: Test various expression patterns and data types
  labels:
    category: Debug
  tags:
    - debug
    - console
    - expressions
    - testing
spec:
  inputs:
    properties:
      testData:
        type: object
        title: Test Data
        description: Object containing various data types for testing
        properties:
          stringValue:
            type: string
            default: "test-string"
          numberValue:
            type: number
            default: 42
          booleanValue:
            type: boolean
            default: true
          arrayValue:
            type: array
            items:
              type: string
            default: ["item1", "item2", "item3"]
          objectValue:
            type: object
            properties:
              nested:
                type: string
                default: "nested-value"
      environment:
        type: string
        title: Environment
        description: Target environment
        enum: ["dev", "staging", "prod"]
        default: "dev"
    required:
      - testData
      - environment
  mirrored:
    processedArray:
      type: string
      value: "{{inputs.testData.arrayValue | join(', ')}}"
    environmentInfo:
      type: string
      value: "Environment: {{inputs.environment}} | IsProduction: {{inputs.environment == 'prod'}}"
  handler:
    type: console
    properties:
      output: |
        === Expression Resolution Testing ===
        
        🔤 String Value: {{inputs.testData.stringValue}}
        🔢 Number Value: {{inputs.testData.numberValue}}
        ✅ Boolean Value: {{inputs.testData.booleanValue}}
        
        📋 Array Value (raw): {{inputs.testData.arrayValue}}
        📋 Array Value (processed): {{inputs.processedArray}}
        
        🏗️ Object Value:
        - Nested Property: {{inputs.testData.objectValue.nested}}
        
        🌍 Environment Info:
        {{inputs.environmentInfo}}
        
        ✨ Expression resolution testing complete.
```

## Configuration Verification

```yaml
apiVersion: core.clrslate.io
kind: Activity
metadata:
  name: debug.activity.verifyConfig
  title: Verify Deployment Configuration
  description: Verify configuration before deployment
  labels:
    category: Debug
  tags:
    - debug
    - console
    - verification
    - deployment
spec:
  inputs:
    properties:
      namespace:
        type: string
        title: Target Namespace
        description: Kubernetes namespace for deployment
        default: default
      image:
        type: string
        title: Container Image
        description: Container image to deploy
      secrets:
        type: object
        format: secret
        title: Application Secrets
        description: Secret containing application configuration
        specifications:
          type: azure.secret.appConfig
      cluster:
        type: object
        format: resource
        title: Target Cluster
        description: Kubernetes cluster for deployment
        specifications:
          type: azure.model.aks
    required:
      - image
      - secrets
      - cluster
  mirrored:
    deploymentContext:
      type: string
      value: |
        Namespace: {{inputs.namespace}}
        Cluster: {{inputs.cluster._name}}
        Image: {{inputs.image}}
        Secrets: {{inputs.secrets._name}}
  handler:
    type: console
    properties:
      output: |
        === Deployment Configuration Verification ===
        
        🎯 Target Environment:
        - Namespace: {{inputs.namespace}}
        - Cluster: {{inputs.cluster._name}}
        - Resource Group: {{inputs.cluster.resourceGroup.name}}
        - Region: {{inputs.cluster.location}}
        
        📦 Application Configuration:
        - Container Image: {{inputs.image}}
        - Secrets Reference: {{inputs.secrets._name}}
        
        🔍 Deployment Context:
        {{inputs.deploymentContext}}
        
        ✅ Configuration verified and ready for deployment.
```

## Expression Pattern Examples

### Resource Expression Access Patterns

```yaml
# Accessing resource properties
spec:
  inputs:
    properties:
      cluster:
        type: object
        format: resource
        specifications:
          type: azure.model.aks
  handler:
    type: console
    properties:
      output: |
        Resource Name: {{inputs.cluster._name}}
        API Version: {{inputs.cluster._apiVersion}}
        Resource Kind: {{inputs.cluster._kind}}
        Cluster Version: {{inputs.cluster.kubernetesVersion}}
        Resource Group: {{inputs.cluster.resourceGroup.name}}
        Location: {{inputs.cluster.location}}
```

### Conditional Display Patterns

```yaml
# Using conditional expressions for environment-specific output
mirrored:
  deploymentWarning:
    type: string
    value: |
      {% if inputs.environment == "prod" %}
      ⚠️  PRODUCTION DEPLOYMENT - Extra validation required!
      {% elif inputs.environment == "staging" %}
      🔄 STAGING DEPLOYMENT - Testing environment
      {% else %}
      🚧 DEVELOPMENT DEPLOYMENT - Development environment
      {% endif %}

handler:
  type: console
  properties:
    output: |
      {{inputs.deploymentWarning}}
      
      Environment: {{inputs.environment}}
      Application: {{inputs.appName}}
```

### Array and Object Processing

```yaml
# Processing arrays and complex objects
mirrored:
  configSummary:
    type: string
    value: |
      {% for key, value in inputs.configuration %}
      - {{key}}: {{value}}
      {% endfor %}
  
handler:
  type: console
  properties:
    output: |
      📋 Configuration Summary:
      {{inputs.configSummary}}
      
      📦 Deployment Targets:
      {% for target in inputs.deploymentTargets %}
      - {{target.name}} ({{target.region}})
      {% endfor %}
```

## Validation Examples

### Successful Validation

```yaml
# Activity that passes all console handler validation rules
apiVersion: core.clrslate.io
kind: Activity
metadata:
  name: valid.activity.consoleExample  # ✅ Valid naming pattern
spec:
  inputs:
    properties:
      message:
        type: string
    required:
      - message  # ✅ Required input defined
  handler:
    type: console  # ✅ Supported handler type
    properties:
      output: "Message: {{inputs.message}}"  # ✅ Valid expression syntax
```

### Common Validation Errors

```yaml
# Examples of validation failures

# ❌ Invalid expression syntax
handler:
  type: console
  properties:
    output: "Value: {inputs.value}"  # Missing second brace

# ❌ Referencing non-existent property
handler:
  type: console
  properties:
    output: "Value: {{inputs.nonExistent}}"  # Property doesn't exist in schema

# ❌ Invalid template logic
handler:
  type: console
  properties:
    output: |
      {% if inputs.environment = "prod" %}  # Should use == not =
      Production environment
      {% endif %}
```

## Performance Considerations

### Optimized Expression Usage

```yaml
# Efficient expression patterns for console handler
mirrored:
  # Pre-compute complex expressions
  summary:
    type: string
    value: "{{inputs.appName}}-{{inputs.version}}-{{inputs.environment}}"

handler:
  type: console
  properties:
    output: |
      # Use pre-computed values
      Deployment Summary: {{inputs.summary}}
      
      # Avoid complex inline expressions
      Status: Ready
```

### Content Size Management

```yaml
# Limit output content size for performance
handler:
  type: console
  properties:
    output: |
      # Keep output concise and focused
      App: {{inputs.appName}}
      Status: {{inputs.status}}
      
      # Avoid excessive detail in console output
```

## Related Documentation

- [Console Handler](../console.md) - Handler specification and configuration
- [Execution Framework](../../execution-framework.md) - Core execution concepts
- [Expression System](../../execution-framework.md#expression-system) - Expression syntax and patterns

## Quick Reference

### Common Patterns

```yaml
# Basic output
output: "Value: {{inputs.property}}"

# Multi-line output
output: |
  Line 1: {{inputs.value1}}
  Line 2: {{inputs.value2}}

# Resource access
output: "Cluster: {{inputs.cluster._name}}"

# Conditional content
output: |
  {% if inputs.environment == "prod" %}
  Production deployment
  {% endif %}
```

### Expression Access
- Inputs: `{{inputs.propertyName}}`
- Resources: `{{inputs.resourceName._name}}`
- Mirrored: `{{inputs.computedProperty}}`
- Conditionals: `{% if condition %}...{% endif %}`
