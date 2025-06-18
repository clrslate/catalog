# NGINX

## Overview

Building blocks for managing NGINX deployments and configurations in Kubernetes. This package provides a comprehensive set of activities for deploying NGINX instances, managing configurations through ConfigMaps, setting up Ingress resources for traffic routing, and maintaining NGINX deployments in Kubernetes environments.

## Components

### Activities

- **Deploy NGINX** (`nginx.deployment.deployNginx`): Deploy an NGINX instance on Kubernetes with ConfigMap support
- **Create ConfigMap for NGINX** (`nginx.configmap.createConfigMap`): Create a ConfigMap with NGINX configuration
- **Update NGINX Deployment** (`nginx.deployment.updateDeployment`): Update an existing NGINX deployment to use a new ConfigMap
- **Create NGINX Ingress Resource** (`nginx.ingress.createIngress`): Create an Ingress resource for routing traffic using NGINX
- **Delete NGINX Deployment** (`nginx.deployment.deleteDeployment`): Delete an NGINX deployment from Kubernetes
- **Delete NGINX Ingress Resource** (`nginx.ingress.deleteIngress`): Delete an Ingress resource managed by NGINX

## Dependencies

### Required

None - this package has no required dependencies.

### Optional

None - this package currently has no optional dependencies.

## Usage

### Basic NGINX Deployment

```yaml
# Create ConfigMap with NGINX configuration
activity: nginx.configmap.createConfigMap
inputs:
  configMapName: "nginx-config"
  namespace: "web"
  nginxConf: |
    server {
        listen 80;
        server_name localhost;
        location / {
            root /usr/share/nginx/html;
            index index.html index.htm;
        }
    }

# Deploy NGINX with the ConfigMap
activity: nginx.deployment.deployNginx
inputs:
  namespace: "web"
  replicas: "3"
  configMapName: "nginx-config"
```

### Ingress Configuration

```yaml
# Create Ingress resource for traffic routing
activity: nginx.ingress.createIngress
inputs:
  ingressName: "web-ingress"
  namespace: "web"
  host: "example.com"
  serviceName: "nginx-service"
  servicePort: "80"
```

### Configuration Updates

```yaml
# Update existing deployment with new ConfigMap
activity: nginx.deployment.updateDeployment
inputs:
  deploymentName: "nginx-deployment"
  namespace: "web"
  configMapName: "nginx-config-v2"
```

### Cleanup Operations

```yaml
# Delete NGINX deployment
activity: nginx.deployment.deleteDeployment
inputs:
  deploymentName: "nginx-deployment"
  namespace: "web"

# Delete Ingress resource
activity: nginx.ingress.deleteIngress
inputs:
  ingressName: "web-ingress"
  namespace: "web"
```

## Configuration

### Required Configuration

All activities require basic Kubernetes configuration:

- **namespace**: Target Kubernetes namespace for resources
- **Kubernetes cluster access**: Valid kubeconfig and cluster connectivity

### NGINX-Specific Configuration

#### Deployment Configuration

- **replicas**: Number of NGINX pod instances (defaults to "1")
- **configMapName**: Reference to ConfigMap containing NGINX configuration
- **deploymentName**: Unique identifier for the NGINX deployment

#### ConfigMap Configuration

- **nginxConf**: Complete NGINX configuration content
- **configMapName**: Unique name for the ConfigMap resource

#### Ingress Configuration

- **host**: Domain name for external access (e.g., "example.com")
- **serviceName**: Kubernetes service to route traffic to
- **servicePort**: Target port on the service

### Authentication

These activities use console handlers for demonstration and validation purposes. In a production environment, you would need:

- Kubernetes cluster access and valid kubeconfig
- Appropriate RBAC permissions for creating/managing:
  - Deployments
  - ConfigMaps
  - Ingress resources
  - Services

## NGINX Configuration Examples

### Basic Web Server

```nginx
server {
    listen 80;
    server_name localhost;

    location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
    }

    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }
}
```

### Reverse Proxy Configuration

```nginx
upstream backend {
    server app1.example.com:3000;
    server app2.example.com:3000;
}

server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### Load Balancer Configuration

```nginx
upstream web_servers {
    server web1.example.com:80 weight=3;
    server web2.example.com:80 weight=2;
    server web3.example.com:80 backup;
}

server {
    listen 80;
    server_name www.example.com;

    location / {
        proxy_pass http://web_servers;
        proxy_set_header Host $host;
        proxy_connect_timeout 30s;
        proxy_send_timeout 30s;
        proxy_read_timeout 30s;
    }
}
```

## Deployment Patterns

### Blue-Green Deployment

1. Create new ConfigMap with updated configuration
2. Update deployment to use new ConfigMap
3. Verify new configuration works correctly
4. Clean up old ConfigMap if needed

### Rolling Updates

1. Update ConfigMap with new configuration
2. Use `updateDeployment` activity to trigger rolling update
3. Monitor deployment status and rollback if necessary

### Multi-Environment Setup

```yaml
# Development environment
namespace: "nginx-dev"
replicas: "1"
host: "dev.example.com"

# Staging environment
namespace: "nginx-staging"
replicas: "2"
host: "staging.example.com"

# Production environment
namespace: "nginx-prod"
replicas: "5"
host: "www.example.com"
```

## Best Practices

### ConfigMap Management

- Use descriptive names with version suffixes (e.g., "nginx-config-v1.2")
- Validate NGINX configuration before creating ConfigMaps
- Keep configurations in version control
- Use environment-specific configurations

### Security Considerations

- Limit NGINX user privileges in container
- Use read-only file systems where possible
- Implement proper SSL/TLS configuration
- Configure security headers and rate limiting

### Resource Management

- Set appropriate resource requests and limits
- Use horizontal pod autoscaling for traffic variations
- Monitor NGINX metrics and logs
- Implement health checks and readiness probes

## Notes

- All activities in this package use console handlers for output and validation
- These activities provide structure and validation for NGINX Kubernetes operations
- For production use, consider implementing corresponding PipelineRef handlers with actual Kubernetes API integration
- Ensure proper Kubernetes cluster access and RBAC permissions before using these activities
- Test NGINX configurations in development environments before production deployment
- Consider using NGINX Ingress Controller for advanced ingress features
