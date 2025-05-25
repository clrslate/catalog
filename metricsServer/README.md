# Kubernetes Metrics Server Package

This ClrSlate package provides simple deployment of Kubernetes Metrics Server via Helm charts.

## Overview

The Metrics Server is a scalable, efficient source of container resource metrics for Kubernetes built-in autoscaling pipelines. This package enables easy deployment and management of Metrics Server across Kubernetes clusters.

## Features

- **Helm-based Deployment**: Uses official Kubernetes SIGs Metrics Server Helm chart
- **Cluster Integration**: Seamless integration with existing Kubernetes clusters
- **Security Optimized**: Production-ready security configurations
- **Minimal Configuration**: Sensible defaults with minimal user input required

## Dependencies

### Required
- `k8s` - Kubernetes cluster operations
- `helm` - Helm chart deployment capabilities

### Optional
- `azure` - Enhanced Azure Kubernetes Service (AKS) integration

## Package Structure

```
metricsServer/
├── metadata.yaml                    # Package metadata and dependencies
├── README.md                        # This documentation
├── activities/
│   ├── deploy-metrics-server.yaml   # Deploy Metrics Server activity
│   └── uninstall-metrics-server.yaml # Uninstall Metrics Server activity
├── resources/
│   └── helmCharts/
│       └── metrics-server.yaml      # Pre-configured Helm chart definition
└── pipelineRefs/
    └── metrics-server-install.yaml  # Enhanced installation pipeline
```

## Usage Examples

### Basic Deployment

Deploy Metrics Server to a cluster:

```yaml
# Use the deploy-metrics-server activity
inputs:
  cluster: "my-aks-cluster"
```

The deployment will automatically:
- **Detect existing installations**: Checks for pre-installed metrics-server (common in AKS)
- **Smart handling**: If existing metrics-server works, skips installation
- **Clean replacement**: If existing metrics-server is broken, removes and reinstalls
- Deploy to `kube-system` namespace (from helm chart resource)
- Use release name `metrics-server` (from helm chart resource)
- Apply security-hardened configuration
- Create a trackable helm release resource

### Uninstalling

Remove Metrics Server from a cluster by selecting the helm release:

```yaml
# Use the uninstall-metrics-server activity
inputs:
  cluster: "my-aks-cluster"
  helmRelease: "metricsServer.release.my-cluster.metrics-server"
```

**Benefits of this approach**:
- Users select from existing releases (no need to remember details)
- Generic helm uninstall operation (consistent across packages)
- Automatic cleanup of all related resources
- Cluster context ensures proper authentication

## Technical Details

### Smart Installation Logic

The package includes intelligent pre-installation detection:

1. **Detection Phase**: Checks for existing metrics-server deployment
2. **Compatibility Check**: Tests if existing installation is functional
3. **Decision Logic**:
   - If no metrics-server exists → Fresh Helm installation
   - If Helm-managed metrics-server exists → Upgrade via Helm
   - If non-Helm metrics-server works → Skip installation (no changes needed)
   - If non-Helm metrics-server broken → Remove and reinstall via Helm

**Common Scenarios**:
- **Azure AKS**: Comes with pre-installed metrics-server that usually works
- **Self-managed clusters**: Typically no pre-installed metrics-server
- **Broken installations**: Automatically cleaned up and replaced

### Helm Chart Configuration

The package uses minimal overrides to the official metrics-server chart:

- **Security Context**: Non-root user, read-only filesystem
- **Arguments**: Optimized for Kubernetes environments
- **Priority Class**: Set as system-critical workload

### Resource Requirements

Default resource configuration:
- CPU: As per chart defaults
- Memory: As per chart defaults
- Priority: system-cluster-critical

## Integration

### With ClrSlate Platform

This package integrates seamlessly with:
- **Azure Package**: For AKS cluster deployment
- **K8s Package**: For cluster operations
- **Helm Package**: For chart management

### With Kubernetes

Provides essential metrics for:
- Horizontal Pod Autoscaler (HPA)
- Vertical Pod Autoscaler (VPA)
- kubectl top commands
- Cluster monitoring solutions

## Troubleshooting

### Common Issues

1. **Metrics Not Available**
   - Wait 1-2 minutes after deployment for metrics collection to start
   - Check pod status: `kubectl get pods -n kube-system -l app.kubernetes.io/name=metrics-server`

2. **Permission Errors**
   - Ensure cluster has appropriate RBAC permissions
   - Metrics Server requires cluster-wide read access

3. **TLS Issues**
   - Package includes kubelet connection configuration for most environments
   - For custom environments, kubelet certificates may need adjustment

### Verification

Verify successful deployment:
```bash
# Check metrics availability
kubectl top nodes
kubectl top pods -A

# Check service status
kubectl get apiservice v1beta1.metrics.k8s.io
```

## Support

For issues and questions:
- Check ClrSlate platform documentation
- Review Kubernetes Metrics Server official documentation
- Contact platform engineering team