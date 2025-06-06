# RabbitMQ

## Overview
The RabbitMQ package provides activities, models, and resources for deploying, managing, and integrating RabbitMQ clusters in Kubernetes and cloud-native environments. It enables message queue provisioning, configuration, and monitoring for scalable distributed systems.

## Components
- **Activities**: Deploy RabbitMQ, Delete RabbitMQ, Scale RabbitMQ
- **Models**: RabbitMQ cluster, credentials
- **Resources**: Helm charts for RabbitMQ, Helm releases for RabbitMQ

## Dependencies
### Required
- `k8s`: Required for Kubernetes resource management and deployment.

### Optional
- `observability`: Enables monitoring and metrics integration for RabbitMQ clusters.

## Usage
Integrate RabbitMQ activities into your ClrSlate pipelines to automate provisioning, scaling, and management. Use provided models to define cluster specs and credentials. Reference Helm charts and releases for custom deployments.

## Configuration
- Set RabbitMQ cluster parameters in the model definition.
- Configure credentials for secure access.
- Optionally enable observability for monitoring.
