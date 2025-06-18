# Keycloak

## Overview

Building blocks for managing Keycloak realms, clients, users, and roles. This package provides a comprehensive set of activities for managing identity and access management through Keycloak, including realm administration, client configuration, user management, and role-based access control.

## Components

### Activities

#### Realm Management

- **Create Realm** (`keycloak.realm.createRealm`): Create a new realm in Keycloak
- **Delete Realm** (`keycloak.realm.deleteRealm`): Delete a realm from Keycloak

#### Client Management

- **Create Client** (`keycloak.client.createClient`): Create a new client in a Keycloak realm
- **Delete Client** (`keycloak.client.deleteClient`): Delete a client from a Keycloak realm

#### User Management

- **Create User** (`keycloak.user.createUser`): Create a new user in a Keycloak realm
- **Delete User** (`keycloak.user.deleteUser`): Delete a user from a Keycloak realm

#### Role Management

- **Create Role** (`keycloak.role.createRole`): Create a new role in a Keycloak realm
- **Delete Role** (`keycloak.role.deleteRole`): Delete a role from a Keycloak realm

## Dependencies

### Required

None - this package has no required dependencies.

### Optional

None - this package currently has no optional dependencies.

## Usage

### Realm Management

```yaml
# Create a new realm
activity: keycloak.realm.createRealm
inputs:
  realmName: "mycompany"
  keycloakUrl: "https://keycloak.company.com"
  authToken: "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJhYmMxMjM..."

# Delete a realm
activity: keycloak.realm.deleteRealm
inputs:
  realmName: "old-realm"
  keycloakUrl: "https://keycloak.company.com"
  authToken: "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJhYmMxMjM..."
```

### Client Management

```yaml
# Create a new client
activity: keycloak.client.createClient
inputs:
  realmName: "mycompany"
  clientId: "web-app"
  keycloakUrl: "https://keycloak.company.com"
  authToken: "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJhYmMxMjM..."

# Delete a client
activity: keycloak.client.deleteClient
inputs:
  realmName: "mycompany"
  clientId: "deprecated-app"
  keycloakUrl: "https://keycloak.company.com"
  authToken: "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJhYmMxMjM..."
```

### User Management

```yaml
# Create a new user
activity: keycloak.user.createUser
inputs:
  realmName: "mycompany"
  username: "john.doe"
  email: "john.doe@company.com"
  temporaryPassword: "TempPass123!"
  keycloakUrl: "https://keycloak.company.com"
  authToken: "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJhYmMxMjM..."

# Delete a user
activity: keycloak.user.deleteUser
inputs:
  realmName: "mycompany"
  username: "inactive.user"
  keycloakUrl: "https://keycloak.company.com"
  authToken: "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJhYmMxMjM..."
```

### Role Management

```yaml
# Create a new role
activity: keycloak.role.createRole
inputs:
  realmName: "mycompany"
  roleName: "admin"
  keycloakUrl: "https://keycloak.company.com"
  authToken: "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJhYmMxMjM..."

# Delete a role
activity: keycloak.role.deleteRole
inputs:
  realmName: "mycompany"
  roleName: "deprecated-role"
  keycloakUrl: "https://keycloak.company.com"
  authToken: "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJhYmMxMjM..."
```

### Complete Identity Setup Workflow

```yaml
# Complete workflow for setting up a new application environment
steps:
  - name: Create Application Realm
    activity: keycloak.realm.createRealm
    inputs:
      realmName: "my-application"
      keycloakUrl: "${KEYCLOAK_URL}"
      authToken: "${KEYCLOAK_ADMIN_TOKEN}"

  - name: Create Web Client
    activity: keycloak.client.createClient
    inputs:
      realmName: "my-application"
      clientId: "web-frontend"
      keycloakUrl: "${KEYCLOAK_URL}"
      authToken: "${KEYCLOAK_ADMIN_TOKEN}"

  - name: Create API Client
    activity: keycloak.client.createClient
    inputs:
      realmName: "my-application"
      clientId: "api-backend"
      keycloakUrl: "${KEYCLOAK_URL}"
      authToken: "${KEYCLOAK_ADMIN_TOKEN}"

  - name: Create Admin Role
    activity: keycloak.role.createRole
    inputs:
      realmName: "my-application"
      roleName: "admin"
      keycloakUrl: "${KEYCLOAK_URL}"
      authToken: "${KEYCLOAK_ADMIN_TOKEN}"

  - name: Create User Role
    activity: keycloak.role.createRole
    inputs:
      realmName: "my-application"
      roleName: "user"
      keycloakUrl: "${KEYCLOAK_URL}"
      authToken: "${KEYCLOAK_ADMIN_TOKEN}"

  - name: Create Admin User
    activity: keycloak.user.createUser
    inputs:
      realmName: "my-application"
      username: "admin"
      email: "admin@company.com"
      temporaryPassword: "AdminTemp123!"
      keycloakUrl: "${KEYCLOAK_URL}"
      authToken: "${KEYCLOAK_ADMIN_TOKEN}"
```

## Configuration

### Required Configuration

All activities require Keycloak connectivity:

- **keycloakUrl**: Base URL of your Keycloak server (e.g., "https://keycloak.company.com")
- **authToken**: Valid Keycloak admin authentication token
- **realmName**: Target realm name for operations

### Authentication

These activities use console handlers for demonstration and validation purposes. In a production environment, you would need:

- Keycloak admin authentication token with appropriate permissions
- Network connectivity to the Keycloak server
- Proper admin privileges for realm, client, user, and role management

### Activity-Specific Configuration

#### Realm Management

- **realmName**: Must be unique across the Keycloak instance
- **Admin Privileges**: Requires master realm admin or appropriate permissions

#### Client Management

- **clientId**: Must be unique within the realm
- **Realm Access**: Requires admin access to the target realm

#### User Management

- **username**: Must be unique within the realm
- **email**: Valid email address for user communication
- **temporaryPassword**: Initial password that user must change on first login

#### Role Management

- **roleName**: Must be unique within the realm
- **Permission Management**: Roles can be assigned to users and clients

## Identity and Access Management Patterns

### Multi-Tenant Architecture

```yaml
# Create isolated realms for different tenants
tenants:
  - name: tenant-a
    realm: "company-a"
    clients: ["web-app", "mobile-app"]

  - name: tenant-b
    realm: "company-b"
    clients: ["portal", "api"]
```

### Environment-Specific Realms

```yaml
# Environment separation
environments:
  - name: development
    realm: "myapp-dev"
    url: "https://keycloak-dev.company.com"

  - name: staging
    realm: "myapp-staging"
    url: "https://keycloak-staging.company.com"

  - name: production
    realm: "myapp-prod"
    url: "https://keycloak.company.com"
```

### Role-Based Access Control

```yaml
# Hierarchical role structure
roles:
  - name: "super-admin"
    description: "Full system access"

  - name: "admin"
    description: "Administrative access"

  - name: "user"
    description: "Standard user access"

  - name: "viewer"
    description: "Read-only access"
```

## Best Practices

### Security Considerations

- Use secure authentication tokens with appropriate expiration
- Implement proper network security between services and Keycloak
- Regular rotation of admin credentials
- Use HTTPS for all Keycloak communications
- Implement proper secret management for tokens

### Realm Management

- Use descriptive realm names that reflect purpose or environment
- Implement consistent naming conventions across environments
- Regular backup of realm configurations
- Document realm purposes and access patterns

### Client Configuration

- Use meaningful client IDs that identify the application
- Configure appropriate client settings for security
- Implement proper redirect URI validation
- Use client secrets for confidential clients

### User Management

- Enforce strong password policies
- Implement user account lifecycle management
- Use temporary passwords for initial user creation
- Configure appropriate user attributes and mappings

### Role Management

- Design role hierarchy that reflects organizational structure
- Use descriptive role names and descriptions
- Implement role composition for complex permission models
- Regular audit of role assignments and permissions

## Integration Patterns

### Application Authentication Flow

1. **Setup Realm**: Create application-specific realm
2. **Configure Clients**: Set up web and API clients
3. **Define Roles**: Create application-specific roles
4. **Provision Users**: Create initial user accounts
5. **Assign Permissions**: Map users to appropriate roles

### DevOps Integration

```yaml
# Automated environment provisioning
deployment_workflow:
  - name: Create Environment Realm
    activity: keycloak.realm.createRealm

  - name: Setup Application Clients
    activity: keycloak.client.createClient

  - name: Configure Service Accounts
    activity: keycloak.user.createUser

  - name: Assign Service Roles
    activity: keycloak.role.createRole
```

### Multi-Application Setup

```yaml
# Shared authentication realm for multiple applications
shared_realm_setup:
  - realm: "corporate"
    clients:
      - "hr-system"
      - "finance-app"
      - "crm-platform"
    roles:
      - "hr-admin"
      - "finance-user"
      - "sales-manager"
```

## Troubleshooting

### Common Issues

- **Authentication failures**: Verify admin token validity and permissions
- **Realm creation errors**: Check naming conflicts and admin privileges
- **Client configuration issues**: Validate client settings and redirect URIs
- **User creation failures**: Check username uniqueness and password policies

### Error Handling

- Implement retry mechanisms for transient failures
- Log detailed error information for debugging
- Provide meaningful error messages for administrators
- Establish escalation procedures for security-related issues

### Monitoring and Maintenance

- Regular health checks of Keycloak instance
- Monitor authentication success and failure rates
- Automated alerting for security events
- Periodic review of user accounts and permissions

## Security Guidelines

### Token Management

- Store admin tokens securely using secrets management
- Rotate authentication tokens regularly
- Use least privilege principle for token permissions
- Monitor token usage and access patterns

### Access Control

- Implement proper realm isolation
- Configure appropriate client access policies
- Use role-based access control effectively
- Regular audit of user permissions and access

### Network Security

- Use HTTPS for all communications
- Implement proper firewall rules
- Configure network isolation where required
- Validate SSL certificates in production

## Notes

- All activities in this package use console handlers for output and validation
- These activities provide structure and validation for Keycloak operations
- For production use, consider implementing corresponding PipelineRef handlers with actual Keycloak API integration
- Ensure proper Keycloak server access and authentication before using these activities
- Consider implementing automated user lifecycle management
- Regular backup and disaster recovery planning for Keycloak data is recommended
