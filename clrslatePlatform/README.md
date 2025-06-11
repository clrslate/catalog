# ClrSlate Platform

## Overview
ClrSlate Platform provides foundational models and activities for multi-tenant, multi-environment ClrSlate deployments. It enables mapping of tenants to environments, environments to clusters and gateways, and supports extensibility for onboarding and management workflows.

## Components
- **Models**:
  - `ClrSlateGateway`: Fully qualified Istio gateway and its wildcard domain
  - `ClrSlateEnvironment`: Environment mapping to AKS cluster and gateway
  - `ClrSlateTenant`: Tenant definition with environment reference and kebab-case slug

## Dependencies
### Required
- None (core platform models)

### Optional
- Integration with `azure` (for AKS clusters)
- Integration with `istio` (for gateway resources)

## Usage
Define gateways, environments, and tenants using the provided models. Reference AKS clusters and gateways as needed for your deployment topology.

## Configuration
- Ensure referenced resources (AKS clusters, gateways) exist in their respective packages.
- Use kebab-case for tenant slugs; this is auto-generated from the tenant name.


