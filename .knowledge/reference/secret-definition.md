---
type: reference
category: core-concepts
complexity: intermediate
prerequisites: [wrapper-model]
outputs: [secret-management-capability, security-understanding]
lastUpdated: 2025-05-28
version: 1.0.0
---

# SecretDefinition

## Overview

SecretDefinition defines secure data schemas with encrypted storage for sensitive information. It provides structured credential management while ensuring security through encryption and access controls.

## Structure

```yaml
apiVersion: core.clrslate.io
kind: SecretDefinition
metadata:
    name: <unique-secret-name>         # Required: Secret schema identifier
    title: <display-title>            # Required: Human-readable name
    description: <description>        # Optional: Purpose description
    icon: <icon-name>                  # Optional: Visual representation
    color: <color-value>               # Optional: UI color
    tags: [<security-tags>]            # Optional: Categorization
    labels:                            # Optional: Organization
        category: <category-name>
        security-level: <level>
spec:
    schema:                            # Required: String-only schema
        properties:                    # Property definitions
            <property-name>:
                type: string           # Must be "string" only
                title: <title>        # Display name
                description: <desc>    # Property purpose
                # String validation constraints only
        required: [<required-props>]   # Required property list
```

## Security Model

### Encryption at Rest
- **All Values Encrypted**: Secret values are encrypted when stored
- **Key Management**: Platform manages encryption keys securely
- **No Plain Text**: Values never stored in plain text
- **Audit Logging**: Access and modification events are logged

### Access Controls
- **Write-Only UI**: Secret values cannot be retrieved through user interfaces
- **Runtime Access**: Values only accessible during workflow execution
- **Permission-Based**: Access controlled through authorization system
- **Scoped Access**: Secrets can be scoped to specific workflows or activities

### Data Protection
- **No Exposure in Logs**: Secret values are redacted from execution logs
- **Memory Protection**: Values cleared from memory after use
- **Transport Security**: Values encrypted during transmission
- **Backup Encryption**: Backup data maintains encryption

## Schema Constraints

### Type Restrictions
SecretDefinition properties are restricted to string types only. All secret values must be base64 encoded to ensure safe storage and transmission:
```yaml
spec:
    schema:
        properties:
            clientId:
                type: string           # ✓ Allowed
                description: Application client identifier
            clientSecret:
                type: string           # ✓ Allowed
                format: password       # Optional format hint
                description: Application client secret (base64 encoded)
            port:
                type: integer          # ✗ Not allowed - only strings
```

### Validation Support
String validation constraints are supported:
```yaml
properties:
    apiKey:
        type: string
        pattern: "^[A-Za-z0-9]{32}$"      # Regex validation
        minLength: 32                      # Length constraints
        maxLength: 32
        description: 32-character API key
    
    endpoint:
        type: string
        format: uri                        # URI format validation
        description: Service endpoint URL
    
    environment:
        type: string
        enum: [development, staging, production]  # Enum validation
        description: Target environment
```

## Common Secret Patterns

### Cloud Provider Credentials
```yaml
# Azure Service Principal
apiVersion: core.clrslate.io
kind: SecretDefinition
metadata:
    name: azure.secret.service-principal
    title: Azure Service Principal
    description: Azure authentication credentials
    icon: Azure.Logo
    labels:
        category: authentication
        provider: azure
spec:
    schema:
        properties:
            clientId:
                type: string
                title: Client ID
                description: Azure application client ID
            clientSecret:
                type: string
                title: Client Secret
                description: Azure application client secret
                format: password
            tenantId:
                type: string
                title: Tenant ID
                description: Azure tenant identifier
        required: [clientId, clientSecret, tenantId]
```

### Database Credentials
```yaml
# Database Connection
apiVersion: core.clrslate.io
kind: SecretDefinition
metadata:
    name: database.secret.connection
    title: Database Credentials
    description: Database authentication credentials
    icon: Generic.Database
    labels:
        category: database
        type: connection
spec:
    schema:
        properties:
            username:
                type: string
                title: Username
                description: Database username
            password:
                type: string
                title: Password
                description: Database password
                format: password
            host:
                type: string
                title: Host
                description: Database host address
                format: hostname
            port:
                type: string
                title: Port
                description: Database port number
                pattern: "^[0-9]{1,5}$"
        required: [username, password, host]
```

### API Authentication
```yaml
# API Key Authentication
apiVersion: core.clrslate.io
kind: SecretDefinition
metadata:
    name: api.secret.key-auth
    title: API Key Authentication
    description: API key-based authentication
    icon: Generic.Robot
    labels:
        category: api
        auth-type: key
spec:
    schema:
        properties:
            apiKey:
                type: string
                title: API Key
                description: Service API key
                pattern: "^[A-Za-z0-9-_]{20,}$"
            endpoint:
                type: string
                title: API Endpoint
                description: Service API endpoint URL
                format: uri
        required: [apiKey, endpoint]
```

## Secret Instance Creation

Secrets are instantiated using the secrets.clrslate.io apiVersion:
```yaml
apiVersion: secrets.clrslate.io
kind: azure.secret.service-principal    # References SecretDefinition name
metadata:
    name: azure.secrets.prod-sp
    title: Production Service Principal
    description: Azure credentials for production environment
    labels:
        environment: production
        owner: platform-team
spec:
    clientId: "MTIzNDU2NzgtMTIzNC0xMjM0LTEyMzQtMTIzNDU2Nzg5MDEy"     # Base64 encoded client ID
    clientSecret: "eW91ci1zZWNyZXQtdmFsdWUtaGVyZQ=="                 # Base64 encoded secret value
    tenantId: "ODc2NTQzMjEtNDMyMS00MzIxLTQzMjEtMjEwOTg3NjU0MzIx"     # Base64 encoded tenant ID
```

**Value Encoding Requirements**:
- All secret values must be base64 encoded in the instance specification during creation
- Base64 encoding is only required for initial secret submission to the platform
- Once received, the ClrSlate platform encrypts and stores values securely using platform-managed encryption
- Platform automatically decodes values before providing them to templates
- Templates receive plain text values for use in workflows

## Template Integration

### Secret Reference in ModelDefinition
```yaml
# ModelDefinition references SecretDefinition
apiVersion: core.clrslate.io
kind: ModelDefinition
metadata:
    name: azure.model.aks-cluster
spec:
    schema:
        properties:
            credentials:
                type: object
                format: secret         # Indicates secret reference
                title: Azure Credentials
                specifications:
                    type: azure.secret.service-principal
```

### Template Access Patterns
```yaml
# In Activity or workflow templates
mirrored:
    azureLogin:
        type: string
        value: |
            az login --service-principal 
            --username {{inputs.credentials.clientId}}
            --password {{inputs.credentials.clientSecret}}
            --tenant {{inputs.credentials.tenantId}}
```

**Template Value Handling**:
- Secret values are automatically base64 decoded by the platform before template execution
- Templates receive plain text values for direct use in commands and configurations
- No manual decoding is required in template expressions
- Values remain encrypted at rest and in transit outside of template execution context

## Design Patterns

### Credential Hierarchies
```yaml
# Base database credentials
apiVersion: core.clrslate.io
kind: SecretDefinition
metadata:
    name: database.secret.base-connection
spec:
    schema:
        properties:
            username: {type: string}
            password: {type: string, format: password}

# PostgreSQL specific credentials
apiVersion: core.clrslate.io
kind: SecretDefinition
metadata:
    name: database.secret.postgres-connection
spec:
    schema:
        properties:
            username: {type: string}
            password: {type: string, format: password}
            host: {type: string, format: hostname}
            port: {type: string, pattern: "^[0-9]{1,5}$", default: "5432"}
            database: {type: string}
        required: [username, password, host, database]
```

### Environment-Specific Secrets
```yaml
# Development credentials (less restrictive)
apiVersion: core.clrslate.io
kind: SecretDefinition
metadata:
    name: api.secret.dev-auth
    labels:
        environment: development
spec:
    schema:
        properties:
            apiKey: 
                type: string
                minLength: 10  # Less restrictive for dev

# Production credentials (more restrictive)
apiVersion: core.clrslate.io
kind: SecretDefinition
metadata:
    name: api.secret.prod-auth
    labels:
        environment: production
spec:
    schema:
        properties:
            apiKey:
                type: string
                pattern: "^[A-Za-z0-9]{32}$"  # Strict format for prod
            rotationPolicy:
                type: string
                enum: [daily, weekly, monthly]
        required: [apiKey, rotationPolicy]
```

## Validation Framework

### Schema Validation
1. **Type Enforcement**: Only string types allowed in properties
2. **Constraint Validation**: String-specific constraints applied
3. **Required Fields**: Required properties must be present
4. **Format Validation**: URI, hostname, email formats supported
5. **Pattern Matching**: Regex patterns enforced

### Security Validation
1. **Value Encryption**: Ensure all values are encrypted at rest
2. **Access Auditing**: Log all access attempts and modifications
3. **Reference Integrity**: Validate secret references in ModelDefinitions
4. **Permission Checking**: Verify access permissions before value retrieval
5. **Usage Tracking**: Monitor secret usage patterns for anomalies

## AI Agent Guidelines

### SecretDefinition Design
1. **Minimal Schema**: Include only necessary properties for security
2. **String Types Only**: Use string type for all properties
3. **Validation Constraints**: Add appropriate validation for each property
4. **Clear Documentation**: Describe property purpose and format requirements
5. **Security Labels**: Use labels to categorize security requirements

### Secret Organization
1. **Domain Grouping**: Group secrets by functional domain or system
2. **Environment Separation**: Separate secrets by environment
3. **Access Scoping**: Design secrets for specific use cases
4. **Naming Consistency**: Follow consistent naming patterns
5. **Documentation**: Clearly document secret purpose and usage

### Integration Planning
1. **Reference Design**: Plan how secrets will be referenced in ModelDefinitions
2. **Template Patterns**: Design for secure template integration
3. **Access Patterns**: Define clear secret access patterns in workflows
4. **Rotation Strategy**: Plan for credential rotation and updates
5. **Audit Requirements**: Consider auditing and compliance needs

### Security Best Practices
1. **Principle of Least Privilege**: Grant minimal necessary access
2. **Regular Rotation**: Implement credential rotation policies
3. **Usage Monitoring**: Monitor secret access patterns
4. **Compliance Alignment**: Ensure secrets meet regulatory requirements
5. **Incident Response**: Plan for secret compromise scenarios
