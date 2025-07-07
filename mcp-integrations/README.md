# MCP Integrations

## Overview
The MCP (Model Context Protocol) package provides integrations and configurations for Model Context Protocol servers, enabling AI-powered development workflows and agent-based automation within ClrSlate environments.

## Components
- **Activities**: 
  - `mcp.testing.testConnection` - Test MCP server connectivity and configuration
  - `mcp.configuration.createIntegration` - Create new MCP integration configurations
- **Models**: 
  - `mcp.model.integration` - Schema for MCP integration configurations
  - `mcp.credential.server-auth` - Authentication credentials for MCP servers
- **Resources**: 
  - `fetch-mcp-server` - Example MCP Fetch server integration

## Features
- MCP server integration configurations
- Standardized schema for MCP integrations
- Support for various MCP server types and endpoints
- Agent-based workflow enablement

## Usage
This package enables integration with MCP servers to enhance AI-powered development capabilities. MCP integrations can be used to:

- Connect to external MCP servers
- Configure AI agents and tools
- Enable context-aware development workflows
- Integrate with various AI service providers

### Available Activities

#### Testing and Validation
- **Test MCP Connection**: Use `mcp.testing.testConnection` activity to validate MCP server connectivity and configuration
- **Create Integration**: Use `mcp.configuration.createIntegration` activity to generate new MCP integration configurations

#### Integration Workflow
1. Create MCP integration configuration using the provided ModelDefinition
2. Test connectivity using the test connection activity
3. Deploy the integration to enable MCP server access

## Configuration
MCP integrations are configured through `McpIntegration` resources that specify:
- Server connection details
- Authentication and security settings
- Integration metadata and descriptions
- UI customization options (icons, colors)

### Example McpIntegration
```yaml
apiVersion: agent.clrslate.io
kind: McpIntegration
metadata:
  name: example-mcp-server
  title: Example MCP Server
  description: An example MCP server integration
  icon: Brands.Mcp
  color: '#52C8EF'
spec:
  server:
    url: https://example-mcp-server.com
```

## Integration with ClrSlate
MCP integrations enhance ClrSlate's AI capabilities by providing access to external Model Context Protocol servers, enabling advanced agent-based workflows and context-aware development tools.
