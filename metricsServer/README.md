# Kubernetes Metrics Server Package

This ClrSlate package provides comprehensive deployment and management of Kubernetes Metrics Server for container resource metrics collection and autoscaling support.

## Overview

The Metrics Server is a scalable, efficient source of container resource metrics for Kubernetes built-in autoscaling pipelines. This package enables intelligent deployment, configuration, and lifecycle management of Metrics Server across Kubernetes clusters with production-ready security and monitoring capabilities.

## Features

- **Intelligent Deployment**: Automatically detects and handles existing metrics-server installations
- **Production Security**: Hardened security configurations with comprehensive access controls
- **Cluster Integration**: Seamless integration with existing Kubernetes clusters and cloud providers
- **High Availability**: Pod disruption budgets and anti-affinity rules for resilient deployments
- **Network Security**: Built-in network policies for traffic isolation and protection
- **Resource Optimization**: Efficient resource allocation and performance tuning
- **Multi-Platform Support**: Compatible with standard Kubernetes, OpenShift, and major cloud providers
- **Monitoring Ready**: Built-in Prometheus integration and comprehensive health checks

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

### Chart Configuration

The package leverages an enterprise-grade metrics-server chart with comprehensive production features:

#### Security Features
- **Enhanced Security Context**: Non-root user (UID 1001), read-only filesystem, dropped capabilities
- **Security Profiles**: SELinux and seccomp profiles for enhanced container security
- **Network Policies**: Built-in network policies for traffic isolation
- **RBAC**: Fine-grained role-based access controls
- **OpenShift Compatibility**: Automatic security context adaptation for OpenShift

#### Production Features
- **Pod Disruption Budget**: Ensures availability during cluster maintenance
- **Anti-Affinity**: Soft pod anti-affinity for better distribution
- **Health Checks**: Comprehensive liveness, readiness, and startup probes
- **Resource Management**: Resource presets and limits for optimal performance
- **Monitoring**: Built-in ServiceMonitor support for Prometheus integration

#### Chart Versions
- **Chart Version**: 7.4.5
- **App Version**: 0.7.2 (metrics-server)
- **Container Image**: metrics-server:0.7.2-debian-12-r24

### Resource Requirements

Default resource configuration:
- **Resource Preset**: nano (optimized for metrics collection)
- **Security Priority**: system-cluster-critical
- **Memory**: Optimized for container metrics aggregation
- **CPU**: Minimal CPU requirements with efficient metric collection

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

## Chart Benefits

This package leverages an enterprise-grade metrics-server chart providing several key advantages:

### Enterprise Features
- **Enhanced Security**: Comprehensive security contexts, network policies, and RBAC
- **Production Readiness**: Pod disruption budgets, health checks, and resource management
- **Multi-Platform Support**: OpenShift compatibility with automatic security adaptation
- **Monitoring Integration**: Built-in Prometheus ServiceMonitor support

### Operational Benefits
- **Optimized Images**: Security-patched and performance-optimized container images
- **Regular Updates**: Frequent security updates and patches
- **Comprehensive Configuration**: Extensive configuration options and examples
- **Active Maintenance**: Well-maintained with community-driven improvements

### Advanced Configuration
The chart provides extensive configuration options including:
- Fine-grained security controls
- Resource management and optimization
- Network policy configurations
- Advanced scheduling options
- Monitoring and observability features

## Support

For issues and questions:
- Check ClrSlate platform documentation
- Review chart documentation and configuration guides
- Review Kubernetes Metrics Server official documentation
- Contact platform engineering team