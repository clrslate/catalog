# Observability Package

Complete observability stack for Kubernetes clusters with monitoring, logging, and visualization capabilities.

## Overview

This package provides a comprehensive observability solution featuring:
- **Prometheus** - Metrics collection and storage
- **Loki** - Log aggregation and storage
- **Promtail** - Log collection agent
- **Grafana** - Visualization and dashboards

## Components

### Monitoring Stack
- Prometheus for metrics collection
- Alert Manager for alerting
- Node Exporter for system metrics
- Kube State Metrics for Kubernetes metrics

### Logging Stack
- Loki for log storage and querying
- Promtail for log collection from Kubernetes pods
- Integration with Prometheus for unified observability

### Visualization
- Grafana for creating dashboards and alerts
- Pre-configured datasources for Prometheus and Loki
- Standard dashboards for Kubernetes monitoring

## Activities

### Deployment Activities
- `observability.activity.deploy-prometheus` - Deploy Prometheus monitoring stack
- `observability.activity.deploy-loki` - Deploy Loki logging stack
- `observability.activity.deploy-promtail` - Deploy Promtail log collectors
- `observability.activity.deploy-grafana` - Deploy Grafana visualization

### Management Activities
- `observability.activity.uninstall-prometheus` - Remove Prometheus stack
- `observability.activity.uninstall-loki` - Remove Loki stack
- `observability.activity.uninstall-promtail` - Remove Promtail agents
- `observability.activity.uninstall-grafana` - Remove Grafana

## Configuration Variants

Each component supports two deployment variants:

### Development Environment
- Minimal resource requirements
- Single-instance deployments
- Short retention periods
- Simplified configurations

### Production Environment
- High availability configurations
- Persistent storage
- Extended retention periods
- Resource limits and monitoring

## Usage

Deploy the complete observability stack:

1. Deploy Prometheus for metrics collection
2. Deploy Loki for log aggregation
3. Deploy Promtail for log collection
4. Deploy Grafana for visualization

Each component can be deployed independently and configured for development or production environments.

## Dependencies

- Kubernetes cluster (AKS supported)
- Helm 3.x
- Persistent storage (for production deployments)

## Package Structure

```
catalog/observability/
├── metadata.yaml                  # Package metadata
├── README.md                     # This documentation
├── activities/                   # Deployment activities
├── models/                      # Configuration models
├── resources/
│   ├── helmCharts/             # Helm chart definitions
│   └── configs/                # Configuration variants