# Helm

## Overview

Building blocks for managing Helm charts and releases in Kubernetes clusters. This package provides comprehensive activities for Helm operations including chart installation, release management, and chart lifecycle operations using Helm CLI and Kubernetes integration.

## Components

### Activities

- **Install Helm Chart** (`helm.activity.install-release`): Install Helm charts in Kubernetes clusters with customizable values
- **Install Helm Release** (`helm.activity.install-release`): Install predefined Helm releases with configured settings
- **Uninstall Helm Release** (`helm.activity.uninstall`): Remove Helm releases from Kubernetes clusters

### Models

- **Helm Chart** (`helm.model.helmChart`): Represents a Helm chart with repository, version, and configuration details
- **Helm Release** (`helm.model.helmRelease`): Represents a Helm release with deployment settings and values

### PipelineRefs

- **Helm Chart Install** (`helm.pipelineRef.helm-chart-install`): Pipeline reference for chart installation operations
- **Helm Uninstall** (`helm.pipelineRef.helm-uninstall`): Pipeline reference for release removal operations

## Dependencies

### Required

None - this package has no required dependencies.

### Optional

- **k8s**: Enhanced integration with Kubernetes cluster operations
- **azure**: Improved support for Azure Kubernetes Service (AKS) clusters

## Usage

### Basic Chart Installation

```yaml
# Install a Helm chart with custom values
activity: helm.activity.install-release
inputs:
  cluster:
    type: azure.model.aks
    name: "my-aks-cluster"
  helmChart:
    type: helm.model.helmChart
    name: "nginx-chart"
  releaseName: "nginx-web"
  namespace: "web-apps"
  createNamespace: true
  values:
    replicaCount: 3
    service:
      type: "LoadBalancer"
    ingress:
      enabled: true
      host: "www.example.com"
```

### Release Management

```yaml
# Install a predefined Helm release
activity: helm.activity.install-release
inputs:
  cluster:
    type: azure.model.aks
    name: "production-cluster"
  helmRelease:
    type: helm.model.helmRelease
    name: "monitoring-stack"

# Uninstall a Helm release
activity: helm.activity.uninstall
inputs:
  cluster:
    type: azure.model.aks
    name: "production-cluster"
  releaseName: "monitoring-stack"
  namespace: "monitoring"
```

### Multi-Environment Deployment

```yaml
# Deploy application across multiple environments
environments:
  - name: development
    activity: helm.activity.install-release
    inputs:
      cluster: "dev-cluster"
      helmChart: "my-app-chart"
      releaseName: "my-app-dev"
      namespace: "development"
      values:
        image:
          tag: "dev-latest"
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"

  - name: production
    activity: helm.activity.install-release
    inputs:
      cluster: "prod-cluster"
      helmChart: "my-app-chart"
      releaseName: "my-app-prod"
      namespace: "production"
      values:
        image:
          tag: "v1.2.3"
        replicaCount: 5
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
```

### Complete Application Stack

```yaml
# Deploy a complete application stack with dependencies
steps:
  - name: Deploy Database
    activity: helm.activity.install-release
    inputs:
      cluster: "${TARGET_CLUSTER}"
      helmChart:
        type: helm.model.helmChart
        name: "postgresql-chart"
      releaseName: "app-database"
      namespace: "data"
      createNamespace: true
      values:
        auth:
          postgresPassword: "${DB_PASSWORD}"
        primary:
          persistence:
            enabled: true
            size: "20Gi"

  - name: Deploy Application
    activity: helm.activity.install-release
    inputs:
      cluster: "${TARGET_CLUSTER}"
      helmChart:
        type: helm.model.helmChart
        name: "my-application-chart"
      releaseName: "my-application"
      namespace: "applications"
      createNamespace: true
      values:
        database:
          host: "app-database-postgresql.data.svc.cluster.local"
          password: "${DB_PASSWORD}"
        ingress:
          enabled: true
          host: "${APP_DOMAIN}"

  - name: Deploy Monitoring
    activity: helm.activity.install-release
    inputs:
      cluster: "${TARGET_CLUSTER}"
      helmRelease:
        type: helm.model.helmRelease
        name: "monitoring-stack"
```

## Configuration

### Required Configuration

All activities require Kubernetes cluster access:

- **cluster**: Target Kubernetes cluster (typically azure.model.aks)
- **helmChart** or **helmRelease**: Chart or release definition to deploy

### Helm Chart Configuration

When using `helmChart` input:

- **Chart Repository**: Chart source (repository URL or local path)
- **Chart Version**: Specific chart version to install
- **Values**: Custom values to override chart defaults
- **Release Name**: Unique name for the Helm release
- **Namespace**: Target Kubernetes namespace

### Authentication

These activities use Tekton pipelines with Kubernetes and Helm integration. You need:

- Kubernetes cluster access with valid credentials
- Helm CLI installed in the execution environment
- Appropriate RBAC permissions for:
  - Creating and managing Kubernetes resources
  - Installing and managing Helm releases
  - Accessing chart repositories

### Activity-Specific Configuration

#### Chart Installation

- **releaseName**: Unique identifier for the Helm release
- **namespace**: Target namespace (created if `createNamespace: true`)
- **values**: Object containing chart value overrides
- **valuesFile**: Path to YAML file containing chart values

#### Release Management

- **helmRelease**: Reference to predefined release configuration
- **createNamespace**: Whether to create namespace if it doesn't exist
- **timeout**: Installation timeout (defaults to Helm default)

## Helm Integration Patterns

### Chart Management

```yaml
# Chart definition with repository and version
helmChart:
  repository: "https://charts.bitnami.com/bitnami"
  name: "nginx"
  version: "13.2.23"
  valuesFile: |
    replicaCount: 3
    service:
      type: LoadBalancer
    resources:
      requests:
        memory: "64Mi"
        cpu: "50m"
```

### Release Templates

```yaml
# Predefined release configuration
helmRelease:
  chart:
    repository: "https://prometheus-community.github.io/helm-charts"
    name: "kube-prometheus-stack"
    version: "45.7.1"
  values:
    prometheus:
      prometheusSpec:
        retention: "30d"
        storageSpec:
          volumeClaimTemplate:
            spec:
              resources:
                requests:
                  storage: "50Gi"
    grafana:
      enabled: true
      adminPassword: "${GRAFANA_ADMIN_PASSWORD}"
```

### GitOps Integration

```yaml
# Version-controlled deployments
deployment_workflow:
  - name: Validate Chart
    # Chart linting and validation

  - name: Deploy to Staging
    activity: helm.activity.install-release
    inputs:
      cluster: "staging-cluster"
      helmChart: "my-app"
      values: "${STAGING_VALUES}"

  - name: Run Tests
    # Automated testing against staging

  - name: Deploy to Production
    activity: helm.activity.install-release
    inputs:
      cluster: "production-cluster"
      helmChart: "my-app"
      values: "${PRODUCTION_VALUES}"
```

## Best Practices

### Chart Management

- Use specific chart versions rather than "latest"
- Pin chart dependencies to stable versions
- Maintain separate values files for different environments
- Validate charts with `helm lint` before deployment

### Release Management

- Use consistent naming conventions for releases
- Implement proper resource limits and requests
- Configure health checks and readiness probes
- Use secrets for sensitive configuration values

### Security Considerations

- Store sensitive values in Kubernetes secrets
- Use RBAC to limit Helm permissions
- Validate chart sources and signatures
- Regular security scanning of deployed images

### Environment Management

- Separate namespaces for different environments
- Use environment-specific value files
- Implement proper backup and disaster recovery
- Monitor resource usage and costs

## Advanced Usage

### Custom Values Management

```yaml
# Template-based values configuration
values:
  global:
    imageRegistry: "${REGISTRY_URL}"
    storageClass: "${STORAGE_CLASS}"

  application:
    name: "${APP_NAME}"
    version: "${APP_VERSION}"

  environment:
    name: "${ENV_NAME}"
    domain: "${ENV_DOMAIN}"

  resources:
    requests:
      memory: "${MEM_REQUEST}"
      cpu: "${CPU_REQUEST}"
    limits:
      memory: "${MEM_LIMIT}"
      cpu: "${CPU_LIMIT}"
```

### Dependency Management

```yaml
# Sequential deployment with dependencies
dependencies:
  - name: Deploy Infrastructure
    activity: helm.activity.install-release
    inputs:
      helmChart: "infrastructure-chart"

  - name: Deploy Database
    activity: helm.activity.install-release
    inputs:
      helmChart: "database-chart"
    depends_on: ["Deploy Infrastructure"]

  - name: Deploy Application
    activity: helm.activity.install-release
    inputs:
      helmChart: "application-chart"
    depends_on: ["Deploy Database"]
```

### Rolling Updates

```yaml
# Blue-green deployment strategy
blue_green_deployment:
  - name: Deploy Green Version
    activity: helm.activity.install-release
    inputs:
      releaseName: "${APP_NAME}-green"
      values:
        image:
          tag: "${NEW_VERSION}"

  - name: Test Green Version
    # Health checks and validation

  - name: Switch Traffic
    # Update ingress or service configuration

  - name: Remove Blue Version
    activity: helm.activity.uninstall
    inputs:
      releaseName: "${APP_NAME}-blue"
```

## Troubleshooting

### Common Issues

- **Chart not found**: Verify chart repository and name
- **Values validation errors**: Check YAML syntax and chart schema
- **Namespace permissions**: Ensure proper RBAC configuration
- **Resource conflicts**: Check for existing resources with same names

### Error Handling

- Implement rollback procedures for failed deployments
- Log detailed error information for debugging
- Use Helm's built-in rollback capabilities
- Establish monitoring and alerting for release status

### Performance Optimization

- Use chart caching for frequently deployed charts
- Optimize values files for faster processing
- Configure appropriate resource limits
- Monitor cluster resource usage during deployments

## Integration with Other Tools

### CI/CD Pipelines

```yaml
# Jenkins/GitHub Actions integration
pipeline_stages:
  - build_and_test
  - package_chart
  - deploy_staging:
      activity: helm.activity.install-release
  - run_integration_tests
  - deploy_production:
      activity: helm.activity.install-release
```

### Monitoring and Observability

```yaml
# Monitoring stack deployment
monitoring_deployment:
  - name: Deploy Prometheus
    activity: helm.activity.install-release
    inputs:
      helmChart: "prometheus"

  - name: Deploy Grafana
    activity: helm.activity.install-release
    inputs:
      helmChart: "grafana"

  - name: Deploy Alert Manager
    activity: helm.activity.install-release
    inputs:
      helmChart: "alertmanager"
```

## Notes

- All activities use Tekton pipelines for execution
- Helm CLI version 3.x is required in the execution environment
- Activities provide structure and validation for Helm operations
- Ensure proper Kubernetes cluster access and RBAC permissions
- Consider implementing Helm chart security scanning
- Regular monitoring of release status and resource usage is recommended
- Use Helm's native rollback capabilities for deployment failures
