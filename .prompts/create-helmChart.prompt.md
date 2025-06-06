---
mode: 'agent'
description: 'Create a new ClrSlate helmChart Record for Helm chart management and deployment'
---

# ClrSlate HelmChart Record Creation Instructions

Your goal is to create a new ClrSlate helmChart Record that conforms to the helm.model.helmChart ModelDefinition for managing Helm chart configurations and enabling template-based deployments.

## Information Gathering

First, collect the following required information from the user:

### Required HelmChart Information
1. **Chart Name**: The name/path of the Helm chart (e.g., "oci://registry.io/mongodb", "bitnami/mongodb")
2. **Chart Version**: Specific version of the chart to use (e.g., "1.0.1", "latest")
3. **Record Instance Name**: Unique identifier for this chart configuration (kebab-case recommended)
4. **Package Domain**: Which ClrSlate package this chart belongs to (e.g., "mongodb", "observability", "platform")
5. **Title**: Human-readable title for the Helm chart record
6. **Description**: Brief description of the chart's purpose and functionality

### Optional Chart Information
- **Repository URL**: Helm chart repository URL (if not OCI registry)
- **Default Namespace**: Target namespace for chart deployment
- **Default Values**: YAML values to override chart defaults
- **Chart Description**: Brief description of chart purpose and functionality

### Context Questions
Ask these questions to gather complete information:

**Chart Source**:
- Is this an OCI registry chart or traditional Helm repository?
- What is the full chart reference (include registry/repository path)?
- Do you need a specific version or should it use latest?

**Deployment Context**:
- What is the primary use case for this chart?
- Will this be deployed to a specific namespace by default?
- Are there common configuration overrides needed?

**Integration Requirements**:
- Will this chart be referenced by HelmRelease Records?
- Does it need to be part of a larger application stack?
- Are there dependencies on other charts or infrastructure?

## HelmChart Record Structure

### ModelDefinition Schema Reference
Based on helm.model.helmChart ModelDefinition:

**Required Properties**:
- `name`: Chart name/path (string)

**Optional Properties**:
- `version`: Chart version (string)
- `repository`: Repository URL (string)
- `namespace`: Default namespace (string)
- `values`: Default values YAML (string)

### Record Creation Template
```yaml
apiVersion: records.clrslate.io
kind: helm.model.helmChart                    # References helm.model.helmChart ModelDefinition
metadata:
  name: <domain>.records.helmChart.<instance-name>   # Required: Qualified record name
  title: <Title>                                      # Required: Human-readable title
  description: <Description>                          # Required: Brief description of chart purpose and functionality
spec:
  name: <chart-name>                          # Required: Chart name/path
  version: <chart-version>                    # Optional: Specific version
  repository: <repository-url>                # Optional: Repository URL
  namespace: <default-namespace>              # Optional: Default namespace
  values: |                                   # Optional: Default values YAML
    <default-values-yaml>
```

## Naming Conventions

### Record Naming Pattern
Follow the standard ClrSlate naming convention:
`<domain>.records.helmChart.<instance-name>`

**Examples**:
- `mongodb.records.helmChart.mongodb-community`
- `observability.records.helmChart.grafana`
- `platform.records.helmChart.cert-manager`
- `istio.records.helmChart.istio-base`

### Chart Name Formats
Support different chart reference formats:

**OCI Registry Charts**:
```yaml
name: oci://clrslatepublic.azurecr.io/helm/mongodb
```

**Traditional Repository Charts**:
```yaml
name: bitnami/mongodb
repository: https://charts.bitnami.com/bitnami
```

**Local or Custom Charts**:
```yaml
name: ./charts/custom-app
```

## Chart Type Categories

### Infrastructure Charts
For foundational platform components:
```yaml
# Example: cert-manager
metadata:
  name: platform.records.helmChart.cert-manager
  title: Cert Manager Helm Chart
  description: Jetstack Cert Manager for Kubernetes certificate management
spec:
  name: cert-manager
  repository: https://charts.jetstack.io
  version: v1.13.0
  namespace: cert-manager
```

### Database Charts
For database and storage systems:
```yaml
# Example: MongoDB
metadata:
  name: mongodb.records.helmChart.mongodb-community
  title: MongoDB Community Helm Chart
  description: Official MongoDB Helm chart for scalable database deployments
spec:
  name: oci://clrslatepublic.azurecr.io/helm/mongodb
  version: 1.0.1
  namespace: mongodb
  values: |
    replicaCount: 3
    persistence:
      enabled: true
```

### Observability Charts
For monitoring and logging:
```yaml
# Example: Grafana
metadata:
  name: observability.records.helmChart.grafana
  title: Grafana Helm Chart
  description: Grafana dashboard and visualization platform
spec:
  name: grafana
  repository: https://grafana.github.io/helm-charts
  version: 6.58.9
  namespace: monitoring
```

### Application Charts
For application deployments:
```yaml
# Example: Custom application
metadata:
  name: app.records.helmChart.user-service
  title: User Service Helm Chart
  description: Custom user service application chart
spec:
  name: oci://myregistry.azurecr.io/apps/user-service
  version: 2.1.0
  namespace: applications
```

## Default Values Configuration

### Values Structure
Provide sensible defaults that can be overridden:

```yaml
values: |
  # Resource configuration
  resources:
    requests:
      memory: "256Mi"
      cpu: "100m"
    limits:
      memory: "512Mi"
      cpu: "500m"
  
  # Replica configuration
  replicaCount: 1
  
  # Service configuration
  service:
    type: ClusterIP
    port: 80
  
  # Ingress configuration
  ingress:
    enabled: false
    className: "nginx"
```

### Environment-Specific Overrides
Consider common override patterns:

```yaml
values: |
  # Production-ready defaults
  replicaCount: 3
  
  # High availability configuration
  podDisruptionBudget:
    enabled: true
    minAvailable: 2
  
  # Security defaults
  securityContext:
    runAsNonRoot: true
    runAsUser: 1001
```

## Template Integration Guidelines

### Record Reference Usage
HelmChart Records are typically referenced by HelmRelease Records:

```yaml
# HelmRelease referencing HelmChart
apiVersion: records.clrslate.io
kind: helm.model.helmRelease
metadata:
  name: mongodb.records.helmRelease.production-mongodb
spec:
  chart: mongodb.records.helmChart.mongodb-community  # Reference to HelmChart Record
  namespace: mongodb-prod
  values: |
    # Environment-specific overrides
```

### Template Access Patterns
When referenced in templates, chart properties become accessible:

```yaml
# Template can access chart properties
helm upgrade {{inputs.chart.name}} \
  --version {{inputs.chart.version}} \
  --repo {{inputs.chart.repository}} \
  --namespace {{inputs.chart.namespace}}
```

## Validation Guidelines

### Required Field Validation
Ensure the following requirements are met:

1. **Chart Name**: Must be a valid Helm chart reference
2. **Version Format**: Should follow semantic versioning when specified
3. **Repository URL**: Must be a valid HTTP/HTTPS URL when specified
4. **Namespace Name**: Must follow Kubernetes naming conventions
5. **Values YAML**: Must be valid YAML syntax when provided

### Best Practices
1. **Version Pinning**: Always specify versions for production charts
2. **Repository Documentation**: Include repository URL for discoverability
3. **Namespace Consistency**: Use consistent namespace naming patterns
4. **Values Documentation**: Comment values YAML for clarity
5. **Security Considerations**: Include security-focused default values

## File Creation Instructions

### File Location
Create the Record file in the appropriate package structure:
```
<package-name>/
└── resources/
    └── helmCharts/
        └── <chart-instance-name>.yaml
```

### File Naming
Use descriptive filenames that match the Record instance name:
- `mongodb-community.yaml`
- `grafana.yaml`
- `cert-manager.yaml`
- `istio-base.yaml`

### Complete Example
Generate a complete working example based on user requirements:

```yaml
apiVersion: records.clrslate.io
kind: helm.model.helmChart
metadata:
  name: mongodb.records.helmChart.mongodb-community
  title: MongoDB Community Helm Chart
  description: Official MongoDB Helm chart for scalable database deployments
spec:
  name: oci://clrslatepublic.azurecr.io/helm/mongodb
  version: 1.0.1
  repository: https://charts.mongodb.com
  namespace: mongodb
  values: |
    # MongoDB configuration
    replicaCount: 3
    
    # Authentication
    auth:
      enabled: true
      
    # Persistence
    persistence:
      enabled: true
      size: 10Gi
      
    # Resources
    resources:
      requests:
        memory: "512Mi"
        cpu: "250m"
      limits:
        memory: "1Gi"
        cpu: "500m"
```

## AI Agent Guidelines

When creating HelmChart Records:

1. **Follow Naming Conventions**: Use qualified naming patterns consistently
2. **Validate Schema Compliance**: Ensure all properties match ModelDefinition schema
3. **Provide Complete Configuration**: Include all relevant optional properties
4. **Consider Integration Context**: Design for template and HelmRelease usage
5. **Security-First Defaults**: Include secure default configurations
6. **Documentation**: Add clear descriptions and comments
7. **Version Management**: Recommend specific versions over latest
8. **Domain Organization**: Place charts in appropriate package domains

The resulting HelmChart Record should be immediately usable in HelmRelease Records and Activity templates for Helm deployment operations.
