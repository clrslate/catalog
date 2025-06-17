---
mode: 'agent'
description: 'Create or update a ClrSlate package with standardized structure and metadata'
---

## ClrSlate Specialized Prompt Compliance Policy

To ensure all ClrSlate packages and components meet organizational standards, strict compliance with specialized prompts is required for all component creation and update operations.

**MANDATORY POLICY:**

- For every component type (activities, PipelineRef resources, models, resources), you must always read and apply the instructions from the relevant specialized prompt before generating or updating any component.
- Do not proceed with creation or modification of any component without referencing the appropriate specialized prompt.
- This policy applies to both new components and updates to existing components.
- Compliance with this policy is required to guarantee:
  - Schema and validation consistency
  - Proper integration and relationships
  - Adherence to naming and structural conventions
  - Backward compatibility and upgrade safety

**Specialized prompts include (but are not limited to):**
- `create-package-activity.prompt.md` (for activities)
- `create-tekton-pipelineref.prompt.md` (for PipelineRef resources)
- `create-model-definition.prompt.md` (for ModelDefinitions)
- Resource-specific prompts as available

**Non-compliance with this policy may result in rejection of the package or component update.**

---

# ClrSlate Package Creation and Update Instructions

Your goal is to create a new ClrSlate package or update an existing one following the standardized structure and metadata requirements.

## Operation Mode Detection

First, determine whether this is a **create** or **update** operation:

### For Create Operations
- Package directory does not exist in the workspace
- User explicitly requests a new package
- No existing metadata.yaml file found

### For Update Operations  
- Package directory already exists in the workspace
- User requests modifications to existing package
- Existing metadata.yaml file is present
- User wants to add new components or modify existing ones

**Always check the workspace for existing package directories before proceeding.**

## Information Gathering

### For Create Operations
Collect the following required information from the user:

#### Required Package Information
1. **Package Name**: Must use kebab-case (e.g., "azure", "helm", "k8s")
2. **Package Description**: Clear description of package purpose and capabilities
3. **Package Domain**: Technology domain (cloud provider, container orchestration, database, etc.)
4. **Component Types**: Which component types will be included (activities, models, resources, etc.)

#### Optional Information
- **Dependencies**: Required and optional package dependencies
- **Version**: Semantic version (defaults to 0.1.0)
- **Owner**: Package maintainer (defaults to "Team ClrSlate")
- **Color**: UI color coding (hex format)
- **Keywords**: Search and discovery keywords

### For Update Operations
First, analyze the existing package:

1. **Read existing metadata.yaml** to understand current configuration
2. **Scan package directory structure** to identify existing components
3. **Identify what needs to be updated**:
   - Adding new components (activities, models, resources)
   - Updating existing components
   - Modifying package metadata
   - Adding or removing dependencies
   - Updating documentation

#### Update-Specific Questions
- **What components need to be added or modified?**
- **Are there new dependencies required?**
- **Should the version be incremented?**
- **Do existing components need updates for compatibility?**
- **Is the package description or title changing?**

## Package Structure Creation

### For Create Operations
Create the following standardized directory structure:

#### Required Files
```
package-name/
├── metadata.yaml              # Package metadata and dependencies
└── README.md                  # Package documentation
```

#### Component Directories (Include as needed)
```
├── activities/                # Activity definitions (.yaml files)
├── models/                    # ModelDefinition and CredentialDefinition files
├── resources/                 # Resource instances and configurations
│   ├── helmCharts/           # Helm chart configurations
│   └── helmReleases/         # Helm release configurations
├── configs/                   # Configuration files and mappings
├── pipelines/                 # Tekton pipeline definitions (.pipeline.yaml)
└── pipelineRefs/             # Tekton PipelineRef handler definitions
```

### For Update Operations
Analyze and extend existing structure:

1. **Preserve existing structure** - Don't modify working components unless explicitly requested
2. **Add missing directories** - Create new component directories as needed
3. **Update selectively** - Only modify files that need changes
4. **Backup considerations** - Be aware that changes will overwrite existing files

#### Update Safety Guidelines
- **Read existing files** before making changes to understand current state
- **Preserve working configurations** unless explicitly asked to change them
- **Add new components** without breaking existing functionality
- **Update version** appropriately based on changes (patch, minor, major)
- **Maintain compatibility** with existing dependencies and references

## Metadata.yaml Management

### For Create Operations
Generate metadata.yaml with the following structure:

```yaml
name: <package-name>                    # Required: kebab-case identifier
version: <semantic-version>             # Required: x.y.z format (default: 0.1.0)
title: <display-title>                  # Required: Human-readable name
description: <package-description>      # Required: Purpose and capabilities
owner: <owner-identifier>               # Required: Maintainer (default: Team ClrSlate)
icon: <icon-name>                       # Required: From predefined icon set

# Optional dependencies
optional_dependencies:                  # Packages that enhance functionality
  - <package-name>
required_dependencies:                  # Packages required for functionality
  - <package-name>

# Optional extended metadata
color: <color-value>                    # UI color coding (hex format)
keywords:                              # Search keywords
  - <keyword>
maintainers:                           # Maintainer details
  - name: <maintainer-name>
```

### For Update Operations
Update existing metadata.yaml carefully:

1. **Read current metadata** to understand existing configuration
2. **Preserve existing values** unless explicitly changing them
3. **Update version appropriately**:
   - **Patch** (x.y.Z): Bug fixes, minor updates
   - **Minor** (x.Y.z): New features, new components
   - **Major** (X.y.z): Breaking changes, major restructuring
4. **Add new dependencies** if new components require them
5. **Update description** if package capabilities have expanded
6. **Maintain compatibility** with existing package consumers

#### Version Update Guidelines
- Adding new activities/models: **Minor version bump**
- Bug fixes or documentation updates: **Patch version bump**  
- Changing existing component interfaces: **Major version bump**
- Adding optional dependencies: **Minor version bump**
- Adding required dependencies: **Major version bump**

## Icon Selection Guidelines

Choose appropriate icon based on package type:

### Technology-Specific Icons
- **Cloud Providers**: Azure.Logo, Amazon.Aws, Google.Gcp
- **Container**: Kubernetes.Logo, Kubernetes.Helm, Brands.Istio
- **Databases**: Brands.MongoDb, Brands.PostgreSql, Brands.MySql, Brands.Redis
- **Version Control**: Brands.GitHub, Brands.GitLab, Brands.Git
- **Languages**: Brands.DotNet, Brands.Python, Brands.Java, Brands.Nodejs

### Generic Icons
- **General Purpose**: Generic.Database, Generic.Streaming, Generic.Messaging, Generic.Stack

## Dependency Management

### Required Dependencies
Use when package **cannot function** without dependency:
- Kubernetes operations require `k8s` package
- Azure operations require `azure` package

### Optional Dependencies
Use when package is **enhanced by** dependency:
- Monitoring capabilities with `observability` package
- DNS management with `externalDns` package

## README.md Management

### For Create Operations
Generate comprehensive README.md with:

```markdown
# <Package Title>

## Overview
<Brief description of package purpose and capabilities>

## Components
- **Activities**: <List key activities if present>
- **Models**: <List key models if present>
- **Resources**: <List key resources if present>

## Dependencies
### Required
<List required dependencies and why they're needed>

### Optional
<List optional dependencies and what they enable>

## Usage
<Basic usage examples and integration patterns>

## Configuration
<Configuration requirements and options>
```

### For Update Operations
Update existing README.md intelligently:

1. **Read existing README** to understand current documentation
2. **Preserve existing sections** that are still relevant
3. **Add new sections** for new components or features
4. **Update component lists** to reflect new activities, models, or resources
5. **Refresh usage examples** if new capabilities are available
6. **Update dependencies section** if dependencies have changed
7. **Maintain consistent formatting** with existing style

#### Documentation Update Guidelines
- **Add new components** to the Components section
- **Update usage examples** to showcase new features
- **Refresh version references** if API changes occurred
- **Add migration notes** for breaking changes
- **Update configuration examples** for new options

## File Naming Conventions

Follow these naming patterns:

### Activities
- `create-<resource-type>.yaml` - Creation operations
- `delete-<resource-type>.yaml` - Deletion operations  
- `deploy-<component>.yaml` - Deployment operations
- `<action>-<target>.yaml` - General pattern

### Models
- `<resource-type>.yaml` - Primary resource models
- `credentials.yaml` - Credential definitions
- `<domain>-<type>.yaml` - Domain-specific models

### Resources
- `helmCharts/<chart-name>.yaml` - Helm chart configurations
- `helmReleases/<release-name>.yaml` - Helm release configurations

### Pipelines
- `<package>-<purpose>.pipeline.yaml` - Pipeline definitions
- `<action>-<target>.yaml` - PipelineRef handlers

## Validation Checklist

### For Create Operations
Ensure the created package meets these requirements:

- [ ] Package name uses kebab-case format
- [ ] metadata.yaml contains all required fields
- [ ] Icon selection matches package domain
- [ ] Dependencies are correctly categorized (required vs optional)
- [ ] README.md provides clear documentation
- [ ] Directory structure follows standard layout
- [ ] File names follow established conventions
- [ ] Version follows semantic versioning (x.y.z)

### For Update Operations
Ensure updates maintain package integrity:

- [ ] Existing functionality is preserved unless explicitly changed
- [ ] Version is incremented appropriately for the type of changes
- [ ] New dependencies are properly categorized
- [ ] Documentation reflects all changes and additions
- [ ] File naming conventions are maintained for new components
- [ ] Breaking changes are documented if version requires major bump
- [ ] Backward compatibility is maintained where possible
- [ ] All existing references and dependencies still work

## Component Creation Integration


## Enforcing Specialized Prompts for Component Creation

When creating or updating package components, you **must** use the specialized prompts for each component type to ensure compliance and consistency. The process for each component type is as follows:


### Activities
- **MANDATORY**: Use `create-package-activity.prompt.md` for all ClrSlate activity creation or updates.
- **Process**: Always read and apply the instructions from this prompt before generating or updating any activity. Do not proceed without referencing the prompt.
- **Integration**: Ensure activities align with package domain and naming conventions as defined in the specialized prompt.
- **Dependencies**: For complex operations, use the prompt to determine if a corresponding PipelineRef resource is required.


### PipelineRef Resources
- **MANDATORY**: Use `create-tekton-pipelineref.prompt.md` for all Tekton PipelineRef resource creation or updates.
- **Process**: Always read and apply the instructions from this prompt before generating or updating any PipelineRef resource. Do not proceed without referencing the prompt.
- **Integration**: Link with activities using tekton.pipelineRef handler as specified by the prompt.
- **Requirements**: Requires Tekton cluster infrastructure; follow all requirements in the specialized prompt.


### Models
- **MANDATORY**: Use `create-model-definition.prompt.md` for all ClrSlate ModelDefinition resource creation or updates.
- **Process**: Always read and apply the instructions from this prompt before generating or updating any ModelDefinition. Do not proceed without referencing the prompt. Ensure that schema properties, validation rules, and computed (mirrored) properties are defined as required by the prompt.
- **Integration**: ModelDefinitions must serve as schemas for Records and enable resource references as described in the prompt.
- **Relationships**: Support resource and secret references with proper type specifications, following the specialized prompt.


### Resources
- **MANDATORY**: Use the specialized prompt for the resource type if available; otherwise, create manually based on package domain requirements and ModelDefinition schemas.
- **Process**: Always read and apply the instructions from the specialized prompt before generating or updating any resource. Do not proceed without referencing the prompt.
- **Follow naming**: Use established file naming conventions.
- **Validate**: Ensure proper resource type definitions and schema compliance as enforced by the specialized prompt.

**Note:** If a specialized prompt exists for a component type, you must use it for both creation and updates. Always read and apply the instructions from the appropriate prompt before generating or modifying any component. Do not create or modify components without referencing the specialized prompt. This ensures all components adhere to ClrSlate standards and best practices, and guarantees compliance and consistency across the package.

## Example Workflow

### For Create Operations
1. **Gather Information**: Ask for package name, description, and domain
2. **Create Structure**: Generate directory layout based on component needs
3. **Generate Metadata**: Create metadata.yaml with appropriate fields
4. **Create Documentation**: Generate comprehensive README.md
5. **Create Components**: Use specialized prompts for activities and PipelineRef resources
6. **Validate**: Ensure all requirements are met and components integrate properly

### For Update Operations
1. **Analyze Existing Package**: Read metadata.yaml and scan directory structure
2. **Identify Changes**: Determine what needs to be added, modified, or removed
3. **Plan Updates**: Consider version impact and dependency changes
4. **Update Components**: Use specialized prompts for new components
5. **Update Metadata**: Increment version and update dependencies as needed
6. **Update Documentation**: Refresh README.md with new capabilities
7. **Validate**: Ensure updates maintain package integrity and compatibility

### Decision Framework
**Choose Create when**:
- Package directory doesn't exist
- Starting completely fresh
- User explicitly requests new package

**Choose Update when**:
- Package directory exists with metadata.yaml
- User wants to add/modify existing package
- Enhancing existing functionality

Ask for the package name and domain if not provided, then follow this structured approach to create a complete, standards-compliant ClrSlate package or safely update an existing one.

When the user requests activities for the package, use the `create-package-activity.prompt.md` to ensure proper execution framework compliance and handler selection.

