# Redis

## Overview

The Redis package provides activities, models, and resources for deploying, managing, and integrating Redis clusters in Kubernetes and cloud-native environments. It enables cache provisioning, configuration, and monitoring for scalable distributed systems.

## Components

- **Activities**: Deploy Redis, Delete Redis, Scale Redis
- **Models**: Redis cluster, credentials
- **Resources**: Helm charts for Redis, Helm releases for Redis

## Dependencies

### Required

- `k8s`: Required for Kubernetes resource management and deployment.

### Optional

- `observability`: Enables monitoring and metrics integration for Redis clusters.

## Usage

Integrate Redis activities into your ClrSlate pipelines to automate provisioning, scaling, and management. Use provided models to define cluster specs and credentials. Reference Helm charts and releases for custom deployments.

## Configuration

- Set Redis cluster parameters in the model definition.
- Configure credentials for secure access.
- Optionally enable observability for monitoring.
