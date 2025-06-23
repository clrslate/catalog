# Istio

## Overview

Building blocks for managing Istio service mesh resources and operations in Kubernetes environments. This package provides comprehensive activities for deploying and configuring Istio components including the base installation, Istiod control plane, gateway resources, and namespace management with automatic sidecar injection.

## Components

### Activities

#### Istio Installation & Management

- **Deploy Istio Base** (`istio.activity.deployIstioBase`): Deploy Istio base components and CRDs using Helm
- **Deploy Istiod** (`istio.activity.deployIstiod`): Deploy Istio control plane (Istiod) using Helm
- **Deploy Istio Gateway** (`istio.activity.deployIstioIstioGateway`): Deploy Istio ingress gateway using Helm
- **Deploy Gateway Resource** (`istio.activity.deployIstioGatewayResource`): Deploy Istio Gateway resource configuration

#### Namespace Management

- **Create Istio Namespaces** (`istio.activity.create-namespaces`): Create Kubernetes namespaces with Istio sidecar injection enabled

### Resources

#### Helm Charts

- **Istio Base Chart** (`istio.records.helmChart.istio-base`): Base Istio components and CRDs
- **Istiod Chart** (`istio.records.helmChart.istiod`): Istio control plane configuration
- **Istio Gateway Chart** (`istio.records.helmChart.istio-gateway`): Istio ingress gateway configuration
- **Gateway Resource Chart** (`istio.records.helmChart.gateway-resource`): Istio Gateway resource definitions

## Dependencies

### Required

- **k8s**: Kubernetes cluster operations and resource management
- **helm**: Helm chart deployment and lifecycle management

### Optional

None - this package requires only the core Kubernetes and Helm capabilities.

## Usage

### Complete Istio Installation

```yaml
# Step 1: Deploy Istio Base (CRDs and base components)
activity: istio.activity.deployIstioBase
inputs:
  cluster:
    type: azure.model.aks
    name: "production-cluster"

# Step 2: Deploy Istiod (Control Plane)
activity: istio.activity.deployIstiod
inputs:
  cluster:
    type: azure.model.aks
    name: "production-cluster"

# Step 3: Deploy Istio Gateway (Ingress)
activity: istio.activity.deployIstioIstioGateway
inputs:
  cluster:
    type: azure.model.aks
    name: "production-cluster"
```

### Namespace Management with Istio

```yaml
# Create namespace with automatic sidecar injection
activity: istio.activity.create-namespaces
inputs:
  cluster:
    type: azure.model.aks
    name: "production-cluster"
  namespace: "my-application"

# Multiple namespaces for microservices
steps:
  - name: Create User Service Namespace
    activity: istio.activity.create-namespaces
    inputs:
      cluster: "${TARGET_CLUSTER}"
      namespace: "user-service"

  - name: Create Order Service Namespace
    activity: istio.activity.create-namespaces
    inputs:
      cluster: "${TARGET_CLUSTER}"
      namespace: "order-service"

  - name: Create Payment Service Namespace
    activity: istio.activity.create-namespaces
    inputs:
      cluster: "${TARGET_CLUSTER}"
      namespace: "payment-service"
```

### Gateway Configuration

```yaml
# Deploy Gateway resource for external traffic
activity: istio.activity.deployIstioGatewayResource
inputs:
  cluster:
    type: azure.model.aks
    name: "production-cluster"
  gatewayName: "application-gateway"
  hosts: ["api.example.com", "app.example.com"]
  ports:
    - port: 80
      protocol: "HTTP"
    - port: 443
      protocol: "HTTPS"
```

### Multi-Environment Istio Setup

```yaml
# Development environment
environments:
  - name: development
    steps:
      - name: Deploy Istio Base
        activity: istio.activity.deployIstioBase
        inputs:
          cluster: "dev-cluster"

      - name: Deploy Istiod
        activity: istio.activity.deployIstiod
        inputs:
          cluster: "dev-cluster"

      - name: Create Application Namespace
        activity: istio.activity.create-namespaces
        inputs:
          cluster: "dev-cluster"
          namespace: "development-apps"

  # Production environment
  - name: production
    steps:
      - name: Deploy Istio Base
        activity: istio.activity.deployIstioBase
        inputs:
          cluster: "prod-cluster"

      - name: Deploy Istiod
        activity: istio.activity.deployIstiod
        inputs:
          cluster: "prod-cluster"

      - name: Deploy Istio Gateway
        activity: istio.activity.deployIstioIstioGateway
        inputs:
          cluster: "prod-cluster"

      - name: Create Production Namespaces
        activity: istio.activity.create-namespaces
        inputs:
          cluster: "prod-cluster"
          namespace: "production-apps"
```

## Configuration

### Required Configuration

All activities require Kubernetes cluster access:

- **cluster**: Target Kubernetes cluster (typically azure.model.aks)
- **namespace**: Target namespace for namespace creation activities

### Installation Order

Istio components must be deployed in the following order:

1. **Istio Base**: Install CRDs and base components first
2. **Istiod**: Deploy control plane after base installation
3. **Istio Gateway**: Deploy ingress gateway after control plane
4. **Gateway Resources**: Configure specific gateway resources after gateway deployment

### Authentication

These activities use Tekton pipelines with Kubernetes and Helm integration. You need:

- Kubernetes cluster access with valid credentials
- Helm CLI installed in the execution environment
- Appropriate RBAC permissions for:
  - Installing cluster-wide Istio components
  - Creating and managing Istio CRDs
  - Managing namespaces and labels
  - Deploying Helm releases

## Istio Service Mesh Patterns

### Microservices Architecture

```yaml
# Service mesh for microservices
microservices_setup:
  - name: Deploy Istio Infrastructure
    steps:
      - activity: istio.activity.deployIstioBase
      - activity: istio.activity.deployIstiod
      - activity: istio.activity.deployIstioIstioGateway

  - name: Setup Service Namespaces
    steps:
      - activity: istio.activity.create-namespaces
        inputs:
          namespace: "frontend"
      - activity: istio.activity.create-namespaces
        inputs:
          namespace: "backend"
      - activity: istio.activity.create-namespaces
        inputs:
          namespace: "data"
```

### Traffic Management

```yaml
# Gateway configuration for traffic routing
traffic_management:
  - name: Deploy Public Gateway
    activity: istio.activity.deployIstioGatewayResource
    inputs:
      gatewayName: "public-gateway"
      hosts: ["*.example.com"]

  - name: Deploy Internal Gateway
    activity: istio.activity.deployIstioGatewayResource
    inputs:
      gatewayName: "internal-gateway"
      hosts: ["*.internal.example.com"]
```

### Security Configuration

```yaml
# Namespace isolation with Istio
security_setup:
  - name: Create Isolated Namespaces
    steps:
      - activity: istio.activity.create-namespaces
        inputs:
          namespace: "secure-apps"
      - activity: istio.activity.create-namespaces
        inputs:
          namespace: "public-apps"
```

## Best Practices

### Installation Best Practices

- Deploy Istio components in the correct order (Base → Istiod → Gateway)
- Use specific Helm chart versions for reproducible deployments
- Validate each component before proceeding to the next
- Monitor resource usage during installation

### Namespace Management

- Enable sidecar injection only for namespaces that need it
- Use consistent naming conventions for Istio-enabled namespaces
- Group related services in the same namespace
- Document which namespaces have Istio injection enabled

### Security Considerations

- Implement proper RBAC for Istio components
- Use TLS for all service-to-service communication
- Configure network policies alongside Istio policies
- Regular security updates for Istio components

### Performance Optimization

- Configure resource limits for Istio components
- Monitor sidecar proxy resource usage
- Optimize Istio configuration for your workload patterns
- Use appropriate load balancing algorithms

## Advanced Features

### Traffic Management

```yaml
# Advanced traffic routing configuration
traffic_policies:
  - virtual_services: "Route traffic based on headers/paths"
  - destination_rules: "Configure load balancing and circuit breakers"
  - service_entries: "Register external services"
  - gateways: "Configure ingress and egress traffic"
```

### Security Policies

```yaml
# Istio security configurations
security_policies:
  - authorization_policies: "Define access control"
  - peer_authentication: "Configure mTLS"
  - request_authentication: "JWT validation"
  - security_policies: "Rate limiting and DDoS protection"
```

### Observability Integration

```yaml
# Monitoring and tracing setup
observability:
  - telemetry: "Configure metrics collection"
  - tracing: "Distributed tracing with Jaeger"
  - logging: "Access logs and audit trails"
  - dashboards: "Grafana dashboards for Istio metrics"
```

## Integration Patterns

### CI/CD Integration

```yaml
# Automated Istio deployment in CI/CD
cicd_workflow:
  - name: Validate Cluster
    # Pre-deployment checks

  - name: Deploy Istio Base
    activity: istio.activity.deployIstioBase

  - name: Deploy Control Plane
    activity: istio.activity.deployIstiod

  - name: Validate Installation
    # Post-deployment validation

  - name: Deploy Applications
    # Application deployments with sidecar injection
```

### Multi-Cluster Setup

```yaml
# Multi-cluster Istio deployment
multi_cluster:
  primary_cluster:
    - activity: istio.activity.deployIstioBase
    - activity: istio.activity.deployIstiod

  remote_clusters:
    - activity: istio.activity.deployIstioBase
    # Additional remote cluster configuration
```

## Troubleshooting

### Common Issues

- **CRD conflicts**: Ensure clean installation order
- **Namespace injection issues**: Verify label configuration
- **Gateway connectivity**: Check LoadBalancer and DNS configuration
- **Control plane issues**: Monitor Istiod logs and resource usage

### Validation Steps

```bash
# Verify Istio installation
kubectl get pods -n istio-system

# Check CRDs
kubectl get crd | grep istio

# Validate sidecar injection
kubectl describe namespace <namespace>

# Check gateway status
kubectl get gateway -A
```

### Performance Monitoring

- Monitor CPU and memory usage of Istio components
- Track sidecar proxy resource consumption
- Monitor network latency and throughput
- Set up alerts for Istio component health

## Upgrade and Maintenance

### Upgrade Procedures

1. **Plan Upgrade**: Review Istio release notes and breaking changes
2. **Backup Configuration**: Export existing Istio configurations
3. **Upgrade Base**: Update base components first
4. **Upgrade Control Plane**: Update Istiod after base
5. **Upgrade Sidecars**: Rolling update application pods
6. **Validate**: Comprehensive testing after upgrade

### Maintenance Tasks

- Regular monitoring of Istio components
- Certificate rotation for mTLS
- Configuration backup and version control
- Performance tuning based on metrics

## Integration with Other Packages

### With Observability Package

```yaml
# Combined Istio and monitoring stack
combined_deployment:
  - name: Deploy Istio
    activities: [istio.activity.deployIstioBase, istio.activity.deployIstiod]

  - name: Deploy Monitoring
    activities:
      [
        observability.activity.deploy-prometheus,
        observability.activity.deploy-grafana,
      ]
```

### With Azure Package

```yaml
# Istio on Azure AKS with Azure integration
azure_integration:
  cluster:
    type: azure.model.aks
    features:
      - istio_service_mesh: true
      - azure_load_balancer: true
      - azure_dns: true
```

## Notes

- All activities use Tekton pipelines with Helm for execution
- Istio requires Kubernetes 1.22+ for optimal functionality
- Activities provide structure and validation for Istio operations
- Ensure proper cluster resources (CPU, memory) before Istio installation
- Consider network policies and security requirements before deployment
- Regular monitoring of Istio control plane and data plane health is recommended
- Keep Istio versions up to date for security and performance improvements
