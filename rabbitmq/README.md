# RabbitMQ

## Overview

Building blocks for deploying, managing, and integrating RabbitMQ message broker clusters in Kubernetes and cloud-native environments. This package provides comprehensive capabilities for message queue provisioning, configuration, monitoring, and scaling for distributed systems requiring reliable message delivery and event-driven architectures.

## Components

### Activities

- **Deploy RabbitMQ** (`rabbitmq.activity.deployRabbitmq`): Deploy RabbitMQ message broker using Helm charts in Kubernetes clusters
- **Scale RabbitMQ** (Future): Scale RabbitMQ cluster instances based on load requirements
- **Delete RabbitMQ** (Future): Remove RabbitMQ deployments and associated resources

### Models

- **RabbitMQ Cluster** (`rabbitmq.cluster`): Represents a RabbitMQ cluster configuration with settings and specifications
- **RabbitMQ Credentials** (`rabbitmq.credentials`): Authentication credentials for secure RabbitMQ access

### Resources

#### Helm Charts

- **RabbitMQ Chart** (`rabbitmq.records.helmChart.rabbitmq`): Pre-configured RabbitMQ Helm chart for message broker deployment
- **RabbitMQ Release** (`rabbitmq.records.helmRelease.rabbitmq`): Helm release configuration for RabbitMQ deployments

## Dependencies

### Required

- **k8s**: Required for Kubernetes resource management and deployment orchestration

### Optional

- **observability**: Enables monitoring and metrics integration for RabbitMQ clusters, providing insights into message throughput, queue depths, and cluster health

## Usage

### Basic RabbitMQ Deployment

```yaml
# Deploy RabbitMQ message broker to a cluster
activity: rabbitmq.activity.deployRabbitmq
inputs:
  cluster:
    type: azure.model.aks
    name: "production-cluster"
  namespace: "messaging"
```

### Multi-Environment RabbitMQ Setup

```yaml
# Development environment
environments:
  - name: development
    activity: rabbitmq.activity.deployRabbitmq
    inputs:
      cluster: "dev-cluster"
      namespace: "dev-messaging"
      replicas: 1
      resources:
        requests:
          memory: "1Gi"
          cpu: "500m"

  - name: staging
    activity: rabbitmq.activity.deployRabbitmq
    inputs:
      cluster: "staging-cluster"
      namespace: "staging-messaging"
      replicas: 3
      resources:
        requests:
          memory: "2Gi"
          cpu: "1000m"

  - name: production
    activity: rabbitmq.activity.deployRabbitmq
    inputs:
      cluster: "prod-cluster"
      namespace: "production-messaging"
      replicas: 5
      persistence:
        enabled: true
        size: "50Gi"
        storageClass: "premium"
```

### Microservices Architecture with RabbitMQ

```yaml
# Event-driven microservices with central message broker
steps:
  - name: Deploy Message Broker
    activity: rabbitmq.activity.deployRabbitmq
    inputs:
      cluster: "${TARGET_CLUSTER}"
      namespace: "messaging-infrastructure"

  - name: Deploy User Service
    # User service that publishes user events
    environment:
      - name: RABBITMQ_URL
        value: "amqp://rabbitmq.messaging-infrastructure.svc.cluster.local:5672"
      - name: RABBITMQ_USERNAME
        valueFrom:
          secretKeyRef:
            name: rabbitmq
            key: username
      - name: RABBITMQ_PASSWORD
        valueFrom:
          secretKeyRef:
            name: rabbitmq
            key: password

  - name: Deploy Order Service
    # Order service that consumes user events and publishes order events
    environment:
      - name: RABBITMQ_URL
        value: "amqp://rabbitmq.messaging-infrastructure.svc.cluster.local:5672"

  - name: Deploy Notification Service
    # Notification service that consumes order events
    environment:
      - name: RABBITMQ_URL
        value: "amqp://rabbitmq.messaging-infrastructure.svc.cluster.local:5672"
```

### High-Availability RabbitMQ Cluster

```yaml
# Production-ready RabbitMQ cluster with high availability
production_cluster:
  activity: rabbitmq.activity.deployRabbitmq
  inputs:
    cluster: "production-cluster"
    namespace: "messaging"
  configuration:
    clustering:
      enabled: true
      addressType: "hostname"
      rebalance: true
    persistence:
      enabled: true
      size: "100Gi"
      storageClass: "premium-ssd"
    auth:
      username: "admin"
      password: "${RABBITMQ_ADMIN_PASSWORD}"
      erlangCookie: "${RABBITMQ_ERLANG_COOKIE}"
    resources:
      requests:
        memory: "4Gi"
        cpu: "2000m"
      limits:
        memory: "8Gi"
        cpu: "4000m"
    metrics:
      enabled: true
      serviceMonitor:
        enabled: true
```

## Configuration

### Required Configuration

All activities require Kubernetes cluster access:

- **cluster**: Target Kubernetes cluster (typically azure.model.aks)
- **namespace**: Target namespace for RabbitMQ deployment

### RabbitMQ Configuration

The RabbitMQ deployment supports various configuration options:

- **Clustering**: Multi-node RabbitMQ cluster for high availability
- **Persistence**: Persistent storage for message durability
- **Authentication**: User credentials and security settings
- **Resources**: CPU and memory allocation for optimal performance

### Authentication

These activities use Tekton pipelines with Kubernetes and Helm integration. You need:

- Kubernetes cluster access with valid credentials
- Helm CLI installed in the execution environment
- Appropriate RBAC permissions for:
  - Creating and managing RabbitMQ deployments
  - Managing persistent volumes for message storage
  - Creating services and networking resources
  - Managing secrets for authentication

## RabbitMQ Deployment Patterns

### Single Instance Deployment

```yaml
# Simple RabbitMQ deployment for development
single_instance:
  activity: rabbitmq.activity.deployRabbitmq
  inputs:
    cluster: "dev-cluster"
    namespace: "development"
  configuration:
    replicas: 1
    persistence:
      enabled: false
    resources:
      requests:
        memory: "512Mi"
        cpu: "250m"
```

### Clustered Deployment

```yaml
# RabbitMQ cluster for production workloads
clustered_deployment:
  activity: rabbitmq.activity.deployRabbitmq
  inputs:
    cluster: "production-cluster"
    namespace: "messaging"
  configuration:
    clustering:
      enabled: true
      replicaCount: 3
    persistence:
      enabled: true
      size: "20Gi"
    auth:
      username: "rabbitmq"
      password: "${SECURE_PASSWORD}"
    podAntiAffinity:
      enabled: true
```

### Multi-Tenant Deployment

```yaml
# Separate RabbitMQ instances per tenant
multi_tenant:
  tenants: ["tenant-a", "tenant-b", "tenant-c"]
  template:
    activity: rabbitmq.activity.deployRabbitmq
    inputs:
      cluster: "multi-tenant-cluster"
      namespace: "messaging-${tenant}"
      isolation: true
```

## Best Practices

### Security Considerations

- Enable authentication and authorization with strong passwords
- Use TLS encryption for client connections
- Implement network policies to restrict broker access
- Regular security updates and patching
- Store credentials securely using Kubernetes secrets

### Performance Optimization

- Configure appropriate resource limits based on expected load
- Use persistent volumes with adequate IOPS for message durability
- Implement proper queue and exchange design patterns
- Monitor message throughput and queue depths
- Configure cluster parameters for optimal performance

### Message Management

- Design appropriate queue and exchange topologies
- Implement message TTL and dead letter queues
- Plan for message retention and cleanup policies
- Monitor disk usage and plan capacity appropriately
- Implement proper error handling and retry mechanisms

### High Availability

- Deploy RabbitMQ clusters with multiple nodes
- Use pod anti-affinity for node distribution
- Configure automated failover and recovery
- Implement proper health checks and readiness probes
- Plan for disaster recovery scenarios

## Advanced Configuration

### Custom RabbitMQ Configuration

```yaml
# Advanced RabbitMQ cluster settings
advanced_config:
  activity: rabbitmq.activity.deployRabbitmq
  inputs:
    cluster: "production-cluster"
    namespace: "messaging"
  values:
    clustering:
      enabled: true
      replicaCount: 5
      addressType: "hostname"
    auth:
      username: "admin"
      password: "${RABBITMQ_PASSWORD}"
      erlangCookie: "${ERLANG_COOKIE}"
    persistence:
      enabled: true
      size: "200Gi"
      storageClass: "premium-ssd"
    configuration: |
      log.console.level = info
      channel_max = 1024
      connection_max = 5000
      heartbeat = 60
      hipe_compile = true
      vm_memory_high_watermark.relative = 0.6
      disk_free_limit.relative = 2.0
    resources:
      requests:
        memory: "8Gi"
        cpu: "4000m"
      limits:
        memory: "16Gi"
        cpu: "8000m"
```

### Load Balancer Configuration

```yaml
# RabbitMQ with external load balancer
load_balanced:
  activity: rabbitmq.activity.deployRabbitmq
  values:
    service:
      type: "LoadBalancer"
      annotations:
        service.beta.kubernetes.io/azure-load-balancer-internal: "true"
    ingress:
      enabled: true
      hostname: "rabbitmq.company.com"
      tls: true
```

### Monitoring and Metrics

```yaml
# RabbitMQ with comprehensive monitoring
monitoring_enabled:
  activity: rabbitmq.activity.deployRabbitmq
  values:
    metrics:
      enabled: true
      serviceMonitor:
        enabled: true
        namespace: "monitoring"
      plugins: "rabbitmq_prometheus"
    extraConfiguration: |
      prometheus.return_per_object_metrics = true
      prometheus.include_aggregated_metrics = true
```

## Integration Patterns

### Application Integration

```yaml
# RabbitMQ integration with applications
application_stack:
  - name: Deploy Message Broker
    activity: rabbitmq.activity.deployRabbitmq
    inputs:
      cluster: "${CLUSTER}"
      namespace: "messaging"

  - name: Deploy Producer Application
    environment:
      - name: RABBITMQ_HOST
        value: "rabbitmq.messaging.svc.cluster.local"
      - name: RABBITMQ_PORT
        value: "5672"
      - name: RABBITMQ_USERNAME
        valueFrom:
          secretKeyRef:
            name: rabbitmq
            key: username
      - name: RABBITMQ_PASSWORD
        valueFrom:
          secretKeyRef:
            name: rabbitmq
            key: password

  - name: Deploy Consumer Application
    # Similar configuration for consumer applications
```

### Event-Driven Architecture

```yaml
# Complete event-driven system with RabbitMQ
event_driven_system:
  - name: Deploy Message Broker
    activity: rabbitmq.activity.deployRabbitmq

  - name: Configure Exchanges and Queues
    # Setup exchanges, queues, and bindings

  - name: Deploy Event Publishers
    # Services that publish domain events

  - name: Deploy Event Consumers
    # Services that consume and process events
```

### Monitoring Integration

```yaml
# RabbitMQ with observability stack
monitoring_integration:
  - name: Deploy RabbitMQ
    activity: rabbitmq.activity.deployRabbitmq

  - name: Configure Monitoring
    # Enable Prometheus metrics and Grafana dashboards
    values:
      metrics:
        enabled: true
        serviceMonitor:
          enabled: true
      management:
        enabled: true
```

## Message Patterns

### Work Queue Pattern

```yaml
# Work queue for task distribution
work_queue:
  exchange: ""
  queue: "task_queue"
  routing_key: "task_queue"
  properties:
    durable: true
    auto_delete: false
```

### Publish/Subscribe Pattern

```yaml
# Fanout exchange for broadcasting
pub_sub:
  exchange: "notifications"
  exchange_type: "fanout"
  queues:
    - "email_notifications"
    - "sms_notifications"
    - "push_notifications"
```

### RPC Pattern

```yaml
# Request/Reply pattern for synchronous communication
rpc_pattern:
  request_queue: "rpc_queue"
  reply_queue: "amq.rabbitmq.reply-to"
  correlation_id: "generated_uuid"
```

### Topic Routing

```yaml
# Topic exchange for complex routing
topic_routing:
  exchange: "events"
  exchange_type: "topic"
  bindings:
    - queue: "user_events"
      routing_key: "user.*"
    - queue: "order_events"
      routing_key: "order.*"
    - queue: "audit_events"
      routing_key: "*.audit"
```

## Troubleshooting

### Common Issues

- **Pod startup failures**: Check resource limits and persistent volume claims
- **Connection issues**: Verify service configuration and network policies
- **Authentication problems**: Check credentials and user permissions
- **Memory issues**: Monitor RabbitMQ memory usage and configure watermarks
- **Clustering problems**: Verify Erlang cookie and network connectivity

### Validation Steps

```bash
# Check RabbitMQ deployment
kubectl get pods -n <namespace> -l app.kubernetes.io/name=rabbitmq

# Check services and endpoints
kubectl get svc,endpoints -n <namespace>

# Check persistent volumes
kubectl get pvc -n <namespace>

# Access RabbitMQ management UI
kubectl port-forward svc/rabbitmq 15672:15672 -n <namespace>
# Open http://localhost:15672

# Check RabbitMQ logs
kubectl logs -f <rabbitmq-pod> -n <namespace>
```

### Performance Monitoring

- Monitor message rates (publish, deliver, acknowledge)
- Track queue depths and consumer utilization
- Monitor memory and disk usage
- Set up alerts for cluster health and connectivity
- Track connection and channel counts

## Scaling and Maintenance

### Horizontal Scaling

```yaml
# Scale RabbitMQ cluster
scaling:
  activity: rabbitmq.activity.deployRabbitmq
  values:
    clustering:
      replicaCount: 7 # Scale to 7 nodes
    resources:
      requests:
        memory: "12Gi"
        cpu: "6000m"
```

### Vertical Scaling

```yaml
# Increase RabbitMQ resources
vertical_scaling:
  values:
    resources:
      requests:
        memory: "16Gi"
        cpu: "8000m"
      limits:
        memory: "32Gi"
        cpu: "16000m"
```

### Maintenance Operations

- Regular backup of RabbitMQ definitions and messages
- Monitor cluster health and node status
- Plan for rolling updates and cluster maintenance
- Capacity planning for message storage and throughput
- Security patch management and version updates

## Integration with Other Packages

### With Observability Package

```yaml
# RabbitMQ with monitoring stack
combined_deployment:
  - name: Deploy RabbitMQ
    activity: rabbitmq.activity.deployRabbitmq

  - name: Deploy Monitoring
    # observability package activities
    values:
      rabbitmq_exporter:
        enabled: true
      grafana_dashboards:
        - rabbitmq_overview
        - rabbitmq_cluster_health
```

### With Azure Package

```yaml
# RabbitMQ on Azure AKS
azure_integration:
  cluster:
    type: azure.model.aks
    features:
      - premium_storage: true
      - load_balancer: true
      - monitoring: true
      - auto_scaling: true
```

## Notes

- All activities use Tekton pipelines with Helm for execution
- RabbitMQ requires careful memory and disk management for optimal performance
- Activities provide structure and validation for RabbitMQ operations
- Ensure proper cluster resources (CPU, memory, storage) before deployment
- Consider message durability and delivery guarantees for your use case
- Regular monitoring of RabbitMQ cluster health and performance is recommended
- Plan for message retention, backup, and disaster recovery scenarios
- Keep RabbitMQ versions up to date for security and performance improvements
- Implement proper message patterns based on your application requirements
