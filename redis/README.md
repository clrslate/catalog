# Redis

## Overview

Building blocks for deploying, managing, and integrating Redis cache clusters in Kubernetes and cloud-native environments. This package provides comprehensive capabilities for cache provisioning, configuration, monitoring, and scaling for distributed systems requiring high-performance in-memory data storage, caching, and session management.

## Components

### Activities

- **Deploy Redis** (`redis.activity.deployRedis`): Deploy Redis cache using Helm charts in Kubernetes clusters
- **Scale Redis** (Future): Scale Redis cluster instances based on load requirements
- **Delete Redis** (Future): Remove Redis deployments and associated resources

### Models

- **Redis Cluster** (`redis.cluster`): Represents a Redis cluster configuration with settings and specifications
- **Redis Credentials** (`redis.credentials`): Authentication credentials for secure Redis access

### Resources

#### Helm Charts

- **Redis Chart** (`redis.records.helmChart.redis`): Pre-configured Redis Helm chart for cache deployment
- **Redis Release** (`redis.records.helmRelease.redis`): Helm release configuration for Redis deployments

## Dependencies

### Required

- **k8s**: Required for Kubernetes resource management and deployment orchestration

### Optional

- **observability**: Enables monitoring and metrics integration for Redis clusters, providing insights into cache hit rates, memory usage, and performance metrics

## Usage

### Basic Redis Deployment

```yaml
# Deploy Redis cache to a cluster
activity: redis.activity.deployRedis
inputs:
  cluster:
    type: azure.model.aks
    name: "production-cluster"
  namespace: "caching"
```

### Multi-Environment Redis Setup

```yaml
# Development environment
environments:
  - name: development
    activity: redis.activity.deployRedis
    inputs:
      cluster: "dev-cluster"
      namespace: "dev-caching"
      replicas: 1
      resources:
        requests:
          memory: "512Mi"
          cpu: "250m"

  - name: staging
    activity: redis.activity.deployRedis
    inputs:
      cluster: "staging-cluster"
      namespace: "staging-caching"
      replicas: 1
      resources:
        requests:
          memory: "1Gi"
          cpu: "500m"

  - name: production
    activity: redis.activity.deployRedis
    inputs:
      cluster: "prod-cluster"
      namespace: "production-caching"
      architecture: "replication"
      master:
        count: 1
      replica:
        replicaCount: 3
      persistence:
        enabled: true
        size: "20Gi"
        storageClass: "premium"
```

### Application Cache Integration

```yaml
# Web application with Redis caching layer
steps:
  - name: Deploy Redis Cache
    activity: redis.activity.deployRedis
    inputs:
      cluster: "${TARGET_CLUSTER}"
      namespace: "cache-layer"

  - name: Deploy Web Application
    # Web application that uses Redis for caching
    environment:
      - name: REDIS_HOST
        value: "redis-master.cache-layer.svc.cluster.local"
      - name: REDIS_PORT
        value: "6379"
      - name: REDIS_PASSWORD
        valueFrom:
          secretKeyRef:
            name: redis
            key: redis-password

  - name: Deploy API Gateway
    # API gateway with Redis rate limiting
    environment:
      - name: REDIS_ENDPOINT
        value: "redis://redis-master.cache-layer.svc.cluster.local:6379"
```

### Microservices with Shared Cache

```yaml
# Multiple microservices sharing Redis cache
microservices:
  - name: User Service
    environment:
      - name: CACHE_REDIS_HOST
        value: "redis-master.shared-cache.svc.cluster.local"
      - name: CACHE_REDIS_DB
        value: "0" # Database 0 for user data

  - name: Session Service
    environment:
      - name: SESSION_STORE_HOST
        value: "redis-master.shared-cache.svc.cluster.local"
      - name: SESSION_STORE_DB
        value: "1" # Database 1 for sessions

  - name: Rate Limiting Service
    environment:
      - name: RATELIMIT_REDIS_HOST
        value: "redis-master.shared-cache.svc.cluster.local"
      - name: RATELIMIT_REDIS_DB
        value: "2" # Database 2 for rate limiting
```

### High-Availability Redis Cluster

```yaml
# Production-ready Redis with replication
production_cluster:
  activity: redis.activity.deployRedis
  inputs:
    cluster: "production-cluster"
    namespace: "caching"
  configuration:
    architecture: "replication"
    auth:
      enabled: true
      password: "${REDIS_PASSWORD}"
    master:
      persistence:
        enabled: true
        size: "50Gi"
        storageClass: "premium-ssd"
      resources:
        requests:
          memory: "4Gi"
          cpu: "2000m"
        limits:
          memory: "8Gi"
          cpu: "4000m"
    replica:
      replicaCount: 3
      persistence:
        enabled: true
        size: "50Gi"
      resources:
        requests:
          memory: "2Gi"
          cpu: "1000m"
    metrics:
      enabled: true
      serviceMonitor:
        enabled: true
```

## Configuration

### Required Configuration

All activities require Kubernetes cluster access:

- **cluster**: Target Kubernetes cluster (typically azure.model.aks)
- **namespace**: Target namespace for Redis deployment

### Redis Configuration

The Redis deployment supports various configuration options:

- **Architecture**: Standalone, replication, or cluster mode
- **Persistence**: Persistent storage for data durability
- **Authentication**: Password-based security
- **Resources**: Memory and CPU allocation for optimal performance

### Authentication

These activities use Tekton pipelines with Kubernetes and Helm integration. You need:

- Kubernetes cluster access with valid credentials
- Helm CLI installed in the execution environment
- Appropriate RBAC permissions for:
  - Creating and managing Redis deployments
  - Managing persistent volumes for data storage
  - Creating services and networking resources
  - Managing secrets for authentication

## Redis Deployment Patterns

### Standalone Deployment

```yaml
# Simple Redis deployment for development
standalone:
  activity: redis.activity.deployRedis
  inputs:
    cluster: "dev-cluster"
    namespace: "development"
  configuration:
    architecture: "standalone"
    persistence:
      enabled: false
    resources:
      requests:
        memory: "256Mi"
        cpu: "100m"
```

### Master-Replica Deployment

```yaml
# Redis with read replicas for high availability
master_replica:
  activity: redis.activity.deployRedis
  inputs:
    cluster: "production-cluster"
    namespace: "caching"
  configuration:
    architecture: "replication"
    master:
      count: 1
      persistence:
        enabled: true
        size: "10Gi"
    replica:
      replicaCount: 2
      persistence:
        enabled: true
        size: "10Gi"
    auth:
      enabled: true
      password: "${SECURE_PASSWORD}"
```

### Cluster Mode Deployment

```yaml
# Redis cluster for horizontal scaling
cluster_mode:
  activity: redis.activity.deployRedis
  inputs:
    cluster: "production-cluster"
    namespace: "caching"
  configuration:
    architecture: "cluster"
    cluster:
      nodes: 6
      replicas: 1
    persistence:
      enabled: true
      size: "20Gi"
    resources:
      requests:
        memory: "2Gi"
        cpu: "1000m"
```

## Best Practices

### Security Considerations

- Enable authentication with strong passwords
- Use TLS encryption for client connections
- Implement network policies to restrict cache access
- Regular security updates and patching
- Store credentials securely using Kubernetes secrets

### Performance Optimization

- Configure appropriate memory limits based on dataset size
- Use persistent volumes with high IOPS for data durability
- Implement proper key expiration policies
- Monitor memory usage and eviction policies
- Configure appropriate maxmemory settings

### Cache Management

- Design efficient key naming conventions
- Implement appropriate TTL (Time To Live) for cached data
- Plan for cache warming and invalidation strategies
- Monitor hit/miss ratios and cache efficiency
- Implement circuit breakers for cache failures

### High Availability

- Deploy Redis in replication mode for read scaling
- Use pod anti-affinity for node distribution
- Configure automated failover mechanisms
- Implement proper health checks and readiness probes
- Plan for disaster recovery and backup scenarios

## Advanced Configuration

### Custom Redis Configuration

```yaml
# Advanced Redis settings
advanced_config:
  activity: redis.activity.deployRedis
  inputs:
    cluster: "production-cluster"
    namespace: "caching"
  values:
    architecture: "replication"
    auth:
      enabled: true
      password: "${REDIS_PASSWORD}"
    master:
      configuration: |
        maxmemory 2gb
        maxmemory-policy allkeys-lru
        timeout 300
        tcp-keepalive 60
        save 900 1
        save 300 10
        save 60 10000
      persistence:
        enabled: true
        size: "50Gi"
        storageClass: "premium-ssd"
      resources:
        requests:
          memory: "4Gi"
          cpu: "2000m"
        limits:
          memory: "8Gi"
          cpu: "4000m"
    replica:
      replicaCount: 3
      configuration: |
        maxmemory 1gb
        maxmemory-policy allkeys-lru
```

### Monitoring and Metrics

```yaml
# Redis with comprehensive monitoring
monitoring_enabled:
  activity: redis.activity.deployRedis
  values:
    metrics:
      enabled: true
      serviceMonitor:
        enabled: true
        namespace: "monitoring"
    master:
      extraFlags:
        - "--latency-monitor-threshold 100"
    sentinel:
      enabled: true
      quorum: 2
```

### Backup and Recovery

```yaml
# Redis with automated backups
backup_enabled:
  activity: redis.activity.deployRedis
  values:
    master:
      persistence:
        enabled: true
        size: "100Gi"
      configuration: |
        save 900 1
        save 300 10
        save 60 10000
        stop-writes-on-bgsave-error yes
        rdbcompression yes
        rdbchecksum yes
```

## Integration Patterns

### Application Integration

```yaml
# Redis integration with applications
application_stack:
  - name: Deploy Cache Layer
    activity: redis.activity.deployRedis
    inputs:
      cluster: "${CLUSTER}"
      namespace: "caching"

  - name: Deploy Application
    environment:
      - name: REDIS_HOST
        value: "redis-master.caching.svc.cluster.local"
      - name: REDIS_PORT
        value: "6379"
      - name: REDIS_PASSWORD
        valueFrom:
          secretKeyRef:
            name: redis
            key: redis-password
      - name: CACHE_TTL
        value: "3600" # 1 hour cache TTL
```

### Session Store Integration

```yaml
# Redis as session store for web applications
session_store:
  - name: Deploy Session Cache
    activity: redis.activity.deployRedis

  - name: Configure Web Application
    environment:
      - name: SESSION_STORE
        value: "redis"
      - name: SESSION_REDIS_HOST
        value: "redis-master.sessions.svc.cluster.local"
      - name: SESSION_REDIS_PREFIX
        value: "sess:"
      - name: SESSION_TIMEOUT
        value: "1800" # 30 minutes
```

### Rate Limiting Integration

```yaml
# Redis for distributed rate limiting
rate_limiting:
  - name: Deploy Rate Limit Cache
    activity: redis.activity.deployRedis

  - name: Configure API Gateway
    # Rate limiting configuration
    limits:
      - name: "user_requests"
        redis_key: "rate_limit:user:${user_id}"
        limit: 100
        window: 60 # requests per minute
```

## Cache Strategies

### Cache-Aside Pattern

```yaml
# Application manages cache explicitly
cache_aside:
  pattern: "cache-aside"
  operations:
    read: "Check cache first, then database if miss"
    write: "Write to database, then invalidate cache"
    update: "Update database, then update/invalidate cache"
```

### Write-Through Pattern

```yaml
# Write to cache and database simultaneously
write_through:
  pattern: "write-through"
  operations:
    write: "Write to cache and database together"
    read: "Read from cache (always up-to-date)"
```

### Write-Behind Pattern

```yaml
# Write to cache immediately, database asynchronously
write_behind:
  pattern: "write-behind"
  operations:
    write: "Write to cache immediately"
    sync: "Periodically sync cache to database"
```

## Data Types and Use Cases

### Key-Value Storage

```yaml
# Simple key-value caching
key_value:
  use_cases:
    - "User profiles"
    - "Configuration data"
    - "API responses"
  operations:
    - "GET/SET for simple values"
    - "EXPIRE for TTL"
```

### Lists and Queues

```yaml
# Redis lists for queues and time series
lists:
  use_cases:
    - "Message queues"
    - "Activity feeds"
    - "Time series data"
  operations:
    - "LPUSH/RPOP for queues"
    - "LRANGE for pagination"
```

### Sets and Sorted Sets

```yaml
# Redis sets for unique collections
sets:
  use_cases:
    - "Tags and categories"
    - "Leaderboards"
    - "Real-time analytics"
  operations:
    - "SADD/SMEMBERS for sets"
    - "ZADD/ZRANGE for sorted sets"
```

### Hashes

```yaml
# Redis hashes for object storage
hashes:
  use_cases:
    - "User sessions"
    - "Shopping carts"
    - "Object attributes"
  operations:
    - "HSET/HGET for fields"
    - "HMGET for multiple fields"
```

## Troubleshooting

### Common Issues

- **Memory issues**: Monitor Redis memory usage and configure maxmemory policies
- **Connection issues**: Verify service configuration and network policies
- **Authentication problems**: Check password configuration and client credentials
- **Performance degradation**: Monitor slow queries and optimize key access patterns
- **Persistence problems**: Check disk space and persistence configuration

### Validation Steps

```bash
# Check Redis deployment
kubectl get pods -n <namespace> -l app.kubernetes.io/name=redis

# Check services and endpoints
kubectl get svc,endpoints -n <namespace>

# Check persistent volumes
kubectl get pvc -n <namespace>

# Connect to Redis CLI
kubectl exec -it <redis-pod> -n <namespace> -- redis-cli

# Monitor Redis performance
kubectl exec -it <redis-pod> -n <namespace> -- redis-cli --latency

# Check Redis info
kubectl exec -it <redis-pod> -n <namespace> -- redis-cli info
```

### Performance Monitoring

- Monitor memory usage and eviction events
- Track cache hit/miss ratios
- Monitor connection counts and command statistics
- Set up alerts for memory thresholds
- Track slow queries and response times

## Scaling and Maintenance

### Horizontal Scaling (Cluster Mode)

```yaml
# Scale Redis cluster
cluster_scaling:
  activity: redis.activity.deployRedis
  values:
    architecture: "cluster"
    cluster:
      nodes: 12 # Scale to 12 nodes
      replicas: 2
```

### Vertical Scaling

```yaml
# Increase Redis resources
vertical_scaling:
  values:
    master:
      resources:
        requests:
          memory: "16Gi"
          cpu: "8000m"
        limits:
          memory: "32Gi"
          cpu: "16000m"
```

### Maintenance Operations

- Regular monitoring of memory usage and performance
- Plan for data backup and recovery procedures
- Monitor persistence and replication lag
- Capacity planning for memory and storage growth
- Security patch management and version updates

## Integration with Other Packages

### With Observability Package

```yaml
# Redis with monitoring stack
combined_deployment:
  - name: Deploy Redis
    activity: redis.activity.deployRedis

  - name: Deploy Monitoring
    # observability package activities
    values:
      redis_exporter:
        enabled: true
      grafana_dashboards:
        - redis_overview
        - redis_performance
```

### With Azure Package

```yaml
# Redis on Azure AKS
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
- Redis requires careful memory management for optimal performance
- Activities provide structure and validation for Redis operations
- Ensure proper cluster resources (CPU, memory) before deployment
- Consider data persistence and backup requirements for your use case
- Regular monitoring of Redis performance and memory usage is recommended
- Plan for cache warming strategies and invalidation policies
- Keep Redis versions up to date for security and performance improvements
- Implement proper cache patterns based on your application requirements
- Consider Redis cluster mode for horizontal scaling beyond single master-replica setup
