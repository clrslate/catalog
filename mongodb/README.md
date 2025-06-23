# MongoDB

## Overview

Building blocks for managing MongoDB resources, models, and activities. This package provides comprehensive capabilities for deploying, configuring, and managing MongoDB NoSQL databases in Kubernetes environments using Helm charts and ClrSlate automation.

## Components

### Activities

- **Deploy MongoDB** (`mongodb.activity.deployMongo`): Deploy MongoDB database using Helm charts in Kubernetes clusters

### Resources

#### Helm Charts

- **MongoDB Chart** (`mongodb.records.helmChart.mongodb`): Pre-configured MongoDB Helm chart for database deployment

## Dependencies

### Required

None - this package has no required dependencies by default.

### Optional

- **k8s**: Enhanced Kubernetes cluster operations and resource management
- **azure**: Improved support for Azure Kubernetes Service (AKS) clusters
- **observability**: Enables monitoring and metrics integration for MongoDB clusters

## Usage

### Basic MongoDB Deployment

```yaml
# Deploy MongoDB to a cluster
activity: mongodb.activity.deployMongo
inputs:
  cluster:
    type: azure.model.aks
    name: "production-cluster"
  namespace: "databases"
```

### Multi-Environment MongoDB Setup

```yaml
# Development environment
environments:
  - name: development
    activity: mongodb.activity.deployMongo
    inputs:
      cluster: "dev-cluster"
      namespace: "dev-databases"

  - name: staging
    activity: mongodb.activity.deployMongo
    inputs:
      cluster: "staging-cluster"
      namespace: "staging-databases"

  - name: production
    activity: mongodb.activity.deployMongo
    inputs:
      cluster: "prod-cluster"
      namespace: "production-databases"
```

### Application Stack with MongoDB

```yaml
# Complete application deployment with MongoDB
steps:
  - name: Deploy MongoDB
    activity: mongodb.activity.deployMongo
    inputs:
      cluster: "${TARGET_CLUSTER}"
      namespace: "data-layer"

  - name: Deploy Application
    # Deploy application that connects to MongoDB
    inputs:
      database:
        host: "mongo-mongodb.data-layer.svc.cluster.local"
        port: "27017"
        name: "myapp"

  - name: Configure Monitoring
    # Optional monitoring setup
    inputs:
      database_type: "mongodb"
      metrics_enabled: true
```

### Microservices with Individual MongoDB Instances

```yaml
# Multiple MongoDB instances for microservices
microservices:
  - name: User Service Database
    activity: mongodb.activity.deployMongo
    inputs:
      cluster: "${CLUSTER}"
      namespace: "user-service"

  - name: Order Service Database
    activity: mongodb.activity.deployMongo
    inputs:
      cluster: "${CLUSTER}"
      namespace: "order-service"

  - name: Inventory Service Database
    activity: mongodb.activity.deployMongo
    inputs:
      cluster: "${CLUSTER}"
      namespace: "inventory-service"
```

## Configuration

### Required Configuration

All activities require Kubernetes cluster access:

- **cluster**: Target Kubernetes cluster (typically azure.model.aks)
- **namespace**: Target namespace for MongoDB deployment

### MongoDB Configuration

The MongoDB deployment uses Helm charts with the following default settings:

- **Release Name**: `mongo` (default)
- **Chart**: Pre-configured MongoDB chart with production-ready settings
- **Namespace**: Specified in activity inputs

### Authentication

These activities use Tekton pipelines with Kubernetes and Helm integration. You need:

- Kubernetes cluster access with valid credentials
- Helm CLI installed in the execution environment
- Appropriate RBAC permissions for:
  - Creating and managing MongoDB deployments
  - Managing persistent volumes for data storage
  - Creating services and networking resources

## MongoDB Deployment Patterns

### Single Instance Deployment

```yaml
# Simple MongoDB deployment for development
single_instance:
  activity: mongodb.activity.deployMongo
  inputs:
    cluster: "dev-cluster"
    namespace: "development"
  configuration:
    replicas: 1
    storage: "10Gi"
    backup: false
```

### High Availability Deployment

```yaml
# Production MongoDB with replica set
production_deployment:
  activity: mongodb.activity.deployMongo
  inputs:
    cluster: "production-cluster"
    namespace: "databases"
  configuration:
    replication:
      enabled: true
      replicas: 3
    persistence:
      enabled: true
      size: "100Gi"
      storageClass: "premium"
    backup:
      enabled: true
      schedule: "0 2 * * *"
```

### Multi-Tenant Deployment

```yaml
# Separate MongoDB instances per tenant
multi_tenant:
  tenants: ["tenant-a", "tenant-b", "tenant-c"]
  template:
    activity: mongodb.activity.deployMongo
    inputs:
      cluster: "multi-tenant-cluster"
      namespace: "tenant-${tenant}"
```

## Best Practices

### Security Considerations

- Enable authentication and authorization
- Use network policies to restrict database access
- Implement proper backup and disaster recovery
- Regular security updates and patching
- Use secrets for database credentials

### Performance Optimization

- Configure appropriate resource limits and requests
- Use persistent volumes with adequate IOPS
- Implement proper indexing strategies
- Monitor database performance metrics
- Configure connection pooling

### Data Management

- Plan data retention and archival strategies
- Implement regular backup procedures
- Use appropriate sharding for large datasets
- Monitor disk usage and plan capacity
- Implement data validation and schema design

### High Availability

- Deploy replica sets for redundancy
- Use anti-affinity rules for pod distribution
- Configure automated failover mechanisms
- Implement proper health checks
- Plan for disaster recovery scenarios

## Advanced Configuration

### Custom MongoDB Configuration

```yaml
# Advanced MongoDB settings
advanced_config:
  activity: mongodb.activity.deployMongo
  inputs:
    cluster: "production-cluster"
    namespace: "databases"
  values:
    auth:
      enabled: true
      rootPassword: "${MONGODB_ROOT_PASSWORD}"
      username: "appuser"
      password: "${MONGODB_APP_PASSWORD}"
      database: "myapp"
    persistence:
      enabled: true
      size: "200Gi"
      storageClass: "ssd"
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

### Replica Set Configuration

```yaml
# MongoDB replica set for high availability
replica_set:
  activity: mongodb.activity.deployMongo
  inputs:
    cluster: "production-cluster"
    namespace: "databases"
  values:
    architecture: "replicaset"
    replicaCount: 3
    auth:
      enabled: true
    persistence:
      enabled: true
      size: "100Gi"
    podAntiAffinity:
      enabled: true
```

### Backup and Recovery

```yaml
# MongoDB with automated backups
backup_enabled:
  activity: mongodb.activity.deployMongo
  inputs:
    cluster: "production-cluster"
    namespace: "databases"
  values:
    backup:
      enabled: true
      cronjob:
        schedule: "0 2 * * *"
        historyLimit: 5
      persistence:
        size: "50Gi"
```

## Integration Patterns

### Application Integration

```yaml
# MongoDB with application deployment
application_stack:
  - name: Deploy Database
    activity: mongodb.activity.deployMongo
    inputs:
      cluster: "${CLUSTER}"
      namespace: "database"

  - name: Deploy Application
    # Application configuration
    environment:
      - name: MONGODB_URI
        value: "mongodb://mongo-mongodb.database.svc.cluster.local:27017/myapp"
      - name: MONGODB_USERNAME
        valueFrom:
          secretKeyRef:
            name: mongo-mongodb
            key: username
      - name: MONGODB_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mongo-mongodb
            key: password
```

### Monitoring Integration

```yaml
# MongoDB with observability stack
monitoring_integration:
  - name: Deploy MongoDB
    activity: mongodb.activity.deployMongo

  - name: Configure Monitoring
    # Enable metrics collection
    values:
      metrics:
        enabled: true
        serviceMonitor:
          enabled: true
          namespace: "monitoring"
```

### Backup Integration

```yaml
# MongoDB with external backup solution
backup_integration:
  database:
    activity: mongodb.activity.deployMongo

  backup_schedule:
    - type: "full"
      schedule: "0 2 * * 0" # Weekly full backup
    - type: "incremental"
      schedule: "0 2 * * 1-6" # Daily incremental
```

## Data Operations

### Database Initialization

```yaml
# Initialize MongoDB with data
initialization:
  - name: Deploy MongoDB
    activity: mongodb.activity.deployMongo

  - name: Initialize Data
    # Run initialization scripts
    scripts:
      - create_indexes.js
      - seed_data.js
      - setup_users.js
```

### Migration Procedures

```yaml
# Database migration workflow
migration:
  - name: Backup Current Data
    # Create backup before migration

  - name: Deploy New MongoDB Version
    activity: mongodb.activity.deployMongo

  - name: Migrate Data
    # Run migration scripts

  - name: Validate Migration
    # Verify data integrity
```

## Troubleshooting

### Common Issues

- **Pod startup failures**: Check resource limits and persistent volume claims
- **Connection issues**: Verify service configuration and network policies
- **Authentication problems**: Check credentials and user permissions
- **Performance issues**: Monitor resource usage and optimize queries

### Validation Steps

```bash
# Check MongoDB deployment
kubectl get pods -n <namespace> -l app.kubernetes.io/name=mongodb

# Check services
kubectl get svc -n <namespace>

# Check persistent volumes
kubectl get pvc -n <namespace>

# Connect to MongoDB
kubectl exec -it <mongodb-pod> -n <namespace> -- mongo
```

### Performance Monitoring

- Monitor CPU and memory usage of MongoDB pods
- Track database connection metrics
- Monitor query performance and slow queries
- Set up alerts for resource thresholds
- Track disk usage and I/O patterns

## Scaling and Maintenance

### Horizontal Scaling

```yaml
# Scale MongoDB replica set
scaling:
  activity: mongodb.activity.deployMongo
  values:
    replicaCount: 5 # Scale to 5 replicas
    resources:
      requests:
        memory: "8Gi"
        cpu: "4000m"
```

### Vertical Scaling

```yaml
# Increase MongoDB resources
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

- Regular backup verification
- Index optimization and maintenance
- Connection monitoring and tuning
- Storage capacity planning
- Security patch management

## Integration with Other Packages

### With Observability Package

```yaml
# MongoDB with monitoring stack
combined_deployment:
  - name: Deploy MongoDB
    activity: mongodb.activity.deployMongo

  - name: Deploy Monitoring
    # observability package activities
    values:
      mongodb_exporter:
        enabled: true
```

### With Azure Package

```yaml
# MongoDB on Azure AKS
azure_integration:
  cluster:
    type: azure.model.aks
    features:
      - premium_storage: true
      - backup_integration: true
      - monitoring: true
```

## Notes

- All activities use Tekton pipelines with Helm for execution
- MongoDB requires persistent storage for data durability
- Activities provide structure and validation for MongoDB operations
- Ensure proper cluster resources (CPU, memory, storage) before deployment
- Consider data protection and compliance requirements
- Regular monitoring of MongoDB performance and resource usage is recommended
- Plan for backup, recovery, and disaster recovery scenarios
- Keep MongoDB versions up to date for security and performance improvements
