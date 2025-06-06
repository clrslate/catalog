---
type: reference
category: core-concepts
complexity: intermediate
prerequisites: [model-definition, wrapper-model]
outputs: [data-instance-management, template-integration]
lastUpdated: 2025-05-28
version: 1.0.0
---

# Records

## Overview

Records are data instances that conform to ModelDefinition schemas. They represent actual configuration data, resource instances, and structured information used throughout ClrSlate workflows and templates.

## Structure

```yaml
apiVersion: records.clrslate.io
kind: <model-definition-name>          # References ModelDefinition name
metadata:
    name: <unique-record-name>         # Required: Instance identifier
    title: <display-title>            # Required: Human-readable name
    description: <description>        # Optional: Instance description
    icon: <icon-name>                  # Optional: Visual representation
    color: <color-value>               # Optional: UI color
    tags: [<instance-tags>]            # Optional: Categorization
    labels:                            # Optional: Organization
        owner: <owner-name>
        environment: <env-name>
spec:
    <property1>: <value1>              # Data conforming to ModelDefinition schema
    <property2>: <value2>              # Property values as defined in schema
    # Additional properties per ModelDefinition
```

## Record-ModelDefinition Relationship

### Schema Conformance
Records must conform to their ModelDefinition schema:
- **Property Validation**: All spec properties validated against schema
- **Required Fields**: Required properties must be present
- **Type Checking**: Values must match defined types
- **Constraint Validation**: Format, pattern, and range constraints enforced

### Kind Resolution
```yaml
# ModelDefinition defines schema
apiVersion: core.clrslate.io
kind: ModelDefinition
metadata:
    name: helm.model.helmChart

# Record uses ModelDefinition name as kind
apiVersion: records.clrslate.io
kind: helm.model.helmChart      # References ModelDefinition name
metadata:
    name: mongo-chart-instance
```

## Automatic Expression Conversion

ClrSlate automatically processes Record properties during instantiation:

### Resource Reference Conversion
Properties with `format: resource` become Expressions:
```yaml
# In Record spec
chart: mongo.records.helmChart.mongodb-chart

# Converted to Expression enabling template access
# Template can use: {{inputs.chart.name}}, {{inputs.chart.version}}
```

### Secret Reference Conversion
Properties with `format: secret` become secure Expressions:
```yaml
# In Record spec
credentials: azure.secrets.service-principal.prod-sp

# Converted to secure Expression for template access
# Template can use: {{inputs.credentials.clientId}}
```

## Template Engine Integration

### Direct Property Access
```yaml
# Record property access
spec:
    name: my-service
    namespace: production

# Template usage
{{inputs.name}}         # → "my-service"
{{inputs.namespace}}    # → "production"
```

### Referenced Record Properties
```yaml
# HelmRelease Record
spec:
    chart: mongo.records.helmChart.mongodb-chart

# Referenced HelmChart Record
spec:
    name: oci://registry.io/mongodb
    version: 1.0.1

# Template access to referenced properties
{{inputs.chart.name}}     # → "oci://registry.io/mongodb"
{{inputs.chart.version}}  # → "1.0.1"
```

### Nested Reference Resolution
```yaml
# Complex reference chain
spec:
    resourceGroup: azure.records.resource-group.production-rg

# ResourceGroup Record references Subscription
spec:
    subscription: azure.records.subscription.production-sub

# Template can traverse full chain
{{inputs.resourceGroup.subscription.id}}
{{inputs.resourceGroup.subscription.credentials.tenantId}}
```

## Naming Conventions

### Standard Format
`<domain>.records.<model-type>.<instance-name>`

**Components**:
- **Domain**: Functional or organizational grouping
- **Model Type**: ModelDefinition name without domain prefix
- **Instance Name**: Specific instance identifier

### Examples by Domain
```yaml
# Infrastructure records
azure.records.aks-cluster.production-cluster
azure.records.resource-group.app-rg
kubernetes.records.namespace.monitoring

# Application records  
app.records.microservice.user-service
app.records.database.user-db
app.records.cache.session-cache

# Platform records
platform.records.secret.database-credentials
platform.records.config.logging-config
helm.records.helmChart.mongodb-chart
```

## Record Types by Use Case

### Configuration Records
Store structured configuration data:
```yaml
apiVersion: records.clrslate.io
kind: app.model.service-config
metadata:
    name: app.records.service-config.user-service
spec:
    serviceName: user-service
    port: 8080
    replicas: 3
    environment: production
```

### Resource Instance Records
Represent actual infrastructure resources:
```yaml
apiVersion: records.clrslate.io
kind: azure.model.aks-cluster
metadata:
    name: azure.records.aks-cluster.production
spec:
    clusterName: prod-aks-01
    resourceGroup: azure.records.resource-group.production-rg
    nodeCount: 3
    vmSize: Standard_D2s_v3
```

### Reference Records
Provide reusable resource references:
```yaml
apiVersion: records.clrslate.io
kind: helm.model.helmChart
metadata:
    name: helm.records.helmChart.mongodb
spec:
    name: oci://clrslatepublic.azurecr.io/helm/mongodb
    version: 1.0.1
    repository: https://charts.mongodb.com
```

## Design Patterns

### Hierarchical Resource Modeling
```yaml
# Subscription Record (top level)
apiVersion: records.clrslate.io
kind: azure.model.subscription
metadata:
    name: azure.records.subscription.production
spec:
    subscriptionId: "12345678-1234-1234-1234-123456789012"
    tenantId: "87654321-4321-4321-4321-210987654321"

# Resource Group Record (references subscription)
apiVersion: records.clrslate.io
kind: azure.model.resource-group
metadata:
    name: azure.records.resource-group.app-rg
spec:
    name: app-production-rg
    location: eastus
    subscription: azure.records.subscription.production

# AKS Cluster Record (references resource group)
apiVersion: records.clrslate.io
kind: azure.model.aks-cluster
metadata:
    name: azure.records.aks-cluster.app-cluster
spec:
    clusterName: app-prod-aks
    resourceGroup: azure.records.resource-group.app-rg
    nodeCount: 3
```

### Service Composition Pattern
```yaml
# Database Record
apiVersion: records.clrslate.io
kind: database.model.instance
metadata:
    name: database.records.instance.user-db
spec:
    host: postgres.production.local
    port: 5432
    databaseName: users
    credentials: database.secrets.postgres.user-db-creds

# Cache Record
apiVersion: records.clrslate.io
kind: cache.model.redis
metadata:
    name: cache.records.redis.session-cache
spec:
    host: redis.production.local
    port: 6379
    credentials: cache.secrets.redis.session-creds

# Service Record (references dependencies)
apiVersion: records.clrslate.io
kind: app.model.microservice
metadata:
    name: app.records.microservice.user-service
spec:
    name: user-service
    version: v1.2.3
    database: database.records.instance.user-db
    cache: cache.records.redis.session-cache
    cluster: azure.records.aks-cluster.app-cluster
```

## Validation Framework

### Schema Validation
Records are validated against their ModelDefinition schema:
1. **Property Presence**: Required properties must exist
2. **Type Validation**: Values must match schema types
3. **Format Validation**: Special formats (resource, secret) enforced
4. **Constraint Checking**: Pattern, range, and enum constraints applied
5. **Reference Integrity**: Referenced Records must exist and be accessible

### Runtime Validation
During template processing:
1. **Reference Resolution**: Ensure referenced Records exist
2. **Property Access**: Validate accessed properties exist in referenced Records
3. **Type Compatibility**: Check template operations match property types
4. **Security Validation**: Verify access permissions for secret references

## AI Agent Guidelines

### Record Creation Strategy
1. **Identify Data Source**: Determine what ModelDefinition schema to use
2. **Design Instance Name**: Create meaningful, unique identifier
3. **Populate Required Properties**: Ensure all required fields are present
4. **Establish References**: Link to dependent Records appropriately
5. **Validate Completeness**: Verify all constraints are satisfied

### Data Organization
1. **Domain Grouping**: Group related Records by functional domain
2. **Naming Consistency**: Follow established naming conventions
3. **Reference Patterns**: Use consistent reference structures
4. **Environment Separation**: Organize Records by environment or lifecycle
5. **Versioning Strategy**: Plan for Record evolution and updates

### Template Integration
1. **Property Design**: Structure data for easy template access
2. **Reference Hierarchy**: Design reference chains for template traversal
3. **Value Computation**: Leverage mirrored properties for complex values
4. **Error Handling**: Ensure robust template property access patterns
5. **Performance**: Minimize deep reference chains where possible

### Relationship Management
1. **Dependency Mapping**: Document Record dependencies clearly
2. **Reference Integrity**: Ensure all references are valid and current
3. **Circular References**: Avoid circular dependency patterns
4. **Cascading Updates**: Plan for changes that affect multiple Records
5. **Cleanup Strategy**: Define Record lifecycle and cleanup procedures
