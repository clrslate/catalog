# ClrSlate Metrics Server Package Design Plan

## 1. Analysis of Existing ClrSlate Patterns

Based on examination of existing packages (azure/, helm/, k8s/, istio/), the following patterns emerge:

### Package Structure Patterns:
- **metadata.yaml**: Package metadata with name, version, title, description, dependencies
- **activities/**: YAML files defining deployable activities 
- **models/**: Resource model definitions using ModelDefinition kind
- **resources/**: Pre-configured resource instances (helmCharts/, helmReleases/)
- **pipelineRefs/**: Tekton pipeline references for execution
- **pipelines/**: Pipeline definitions (optional)
- **configs/**: Configuration files and mappings

### Naming Conventions:
- Package names: lowercase (e.g., `helm`, `istio`, `k8s`)
- Activity names: `[package].activity.[action-description]`
- Model names: `[package].model.[resource-type]`
- Resource records: `[package].records.[type].[name]`
- Pipeline refs: `[package].pipelineRef.[operation]`

### Dependencies:
- Packages commonly depend on `k8s` and `helm` for Kubernetes workloads
- Activities reference cluster resources (typically `azure.model.aks`)
- Helm-based deployments use `helm.pipelineRef.helm-install`

## 2. Metrics Server Package Design

### 2.1 Package Metadata Structure
```
metricsServer/
├── metadata.yaml
├── README.md
├── activities/
│   ├── deploy-metrics-server.yaml
│   └── uninstall-metrics-server.yaml
├── models/
│   ├── metricsServerConfig.yaml
│   └── metricsServerDeployment.yaml
├── resources/
│   └── helmCharts/
│       └── metrics-server.yaml
├── pipelineRefs/
│   └── metrics-server-install.yaml
└── configs/
    └── default-values.yaml
```

### 2.2 Dependencies Analysis
The metricsServer package should depend on:
- **Required**: `k8s` (for cluster operations)
- **Required**: `helm` (for Helm chart deployment)
- **Optional**: `azure` (for Azure-specific cluster integration)

## 3. Detailed Component Design

### 3.1 metadata.yaml
```yaml
name: metricsServer
version: 0.1.0
title: Kubernetes Metrics Server
description: ClrSlate package for deploying and managing Kubernetes Metrics Server via Helm
owner: Team ClrSlate
icon: Kubernetes.Metrics
color: '#326ce5'
keywords:
  - kubernetes
  - metrics
  - monitoring
  - helm
maintainers:
  - name: ClrSlate Team
    email: team@clrslate.io
dependencies:
  - k8s
  - helm
optional_dependencies:
  - azure
```

### 3.2 Activities Design

#### deploy-metrics-server.yaml
- **Purpose**: Deploy Metrics Server using Helm chart
- **Inputs**: cluster, namespace (optional), custom values (optional)
- **Handler**: Uses `helm.pipelineRef.helm-install`
- **Outputs**: Creates metricsServerDeployment resource record

#### uninstall-metrics-server.yaml  
- **Purpose**: Remove Metrics Server deployment
- **Inputs**: cluster, release name, namespace
- **Handler**: Uses kubectl/helm uninstall commands

### 3.3 Models Design

#### metricsServerConfig.yaml
- **Purpose**: Configuration model for Metrics Server settings
- **Properties**: 
  - replicas, resource limits/requests
  - securityContext settings
  - nodeSelector, tolerations
  - metrics collection intervals

#### metricsServerDeployment.yaml
- **Purpose**: Represents deployed Metrics Server instance
- **Properties**: 
  - cluster reference, namespace, release name
  - configuration reference, status
- **Relations**: Links to cluster and config models

### 3.4 Resources Design

#### helmCharts/metrics-server.yaml
- **Chart**: `metrics-server` from `https://kubernetes-sigs.github.io/metrics-server/`
- **Version**: Latest stable (1.0.x)
- **Default namespace**: `kube-system`
- **Values**: Basic configuration for most environments

### 3.5 Pipeline Integration

#### metrics-server-install.yaml
- **Type**: PipelineRef extending helm install capabilities
- **Features**: 
  - Validates cluster readiness
  - Creates namespace if needed
  - Applies Metrics Server-specific configurations
  - Verifies deployment success

## 4. Integration Points

### 4.1 With K8s Package
- Uses `k8s.model.cluster` for cluster references
- Leverages `k8s.pipelineRef.kubectl-script` for verification commands
- Creates resources in `kube-system` namespace (standard for Metrics Server)

### 4.2 With Helm Package  
- Extends `helm.model.helmChart` for chart definition
- Uses `helm.pipelineRef.helm-install` for deployment
- Follows Helm release management patterns

### 4.3 With Azure Package
- Compatible with `azure.model.aks` clusters
- Uses Azure pipeline execution context
- Supports Azure-specific authentication

## 5. Advanced Features

### 5.1 Configuration Management
- Default values optimized for production use
- Support for custom values injection
- Environment-specific configuration templates

### 5.2 Validation & Health Checks
- Pre-deployment cluster readiness validation
- Post-deployment metrics availability verification
- Integration with cluster monitoring systems

### 5.3 Security Considerations
- RBAC configuration for metrics collection
- Secure communication with kubelet
- Resource isolation and limits

## 6. Implementation Roadmap

### Phase 1: Core Package Structure
1. Create basic package structure with metadata
2. Implement core deployment activity
3. Define basic models and Helm chart resource

### Phase 2: Advanced Configuration
1. Add configuration models and validation
2. Implement uninstall activity
3. Create custom pipeline refs for enhanced functionality

### Phase 3: Integration & Testing
1. Validate integration with existing packages
2. Test with different cluster types (AKS, etc.)
3. Performance and security validation

### Phase 4: Documentation & Examples
1. Complete README with usage examples
2. Create sample configurations
3. Integration guides for common scenarios

## 7. File Templates Ready for Implementation

The design provides clear specifications for each component following ClrSlate patterns:
- Consistent naming conventions
- Proper resource relationships
- Standard Tekton pipeline integration
- Helm chart management best practices
- Security and configuration considerations

This design ensures the metricsServer package integrates seamlessly with the existing ClrSlate ecosystem while providing robust Metrics Server deployment capabilities.