# Azure

## Overview

Azure resource models and integrations

## Installation

To install this package, use the ClrSlate CLI:

```bash
clrslate package install azure
```

## Basic Usage

This package provides models for managing resources in ClrSlate.

## Models

### Resource Models

- **Subscription** (`azure.subscription`) - Represents an Azure Subscription.
- **Resource Group** (`azure.resourceGroup`) - Represents an Azure Resource Group.
- **AKS Cluster** (`azure.aks`) - Represents an Azure Kubernetes Service cluster. - Implements `k8s.clusterProvider`
- **Key Vault** (`azure.keyVault`) - Represents an Azure Key Vault. - Implements `secrets.vaultProvider`
- **DNS Zone** (`azure.dnsZone`) - Represents an Azure DNS Zone. - Implements `externalDns.provider`
- **Container Registry** (`azure.containerRegistry`) - Represents an Azure Container Registry.
- **Cosmos DB** (`azure.cosmos`) - Represents an Azure Cosmos DB instance.

### Secret Definitions

- **Azure Credentials** (`azure.credentials`) - Azure service principal credentials (clientId, clientSecret, subscriptionId, tenantId).

## Configuration

To configure this package, you will need:

1. **Azure Subscription** - An active Azure subscription
2. **Service Principal** - An Azure AD Service Principal with appropriate permissions
3. **Authentication Details**:
   - Client ID (Application ID)
   - Client Secret
   - Tenant ID
   - Subscription ID

These credentials should be configured in your ClrSlate environment to allow the package to manage Azure resources.

## License Information

This package is licensed under the MIT License. See the LICENSE file for details.
