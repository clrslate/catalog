# Templates by Complexity

This index will organize ClrSlate templates by complexity level once templates are created based on the actual ClrSlate platform documentation.

**REQUIREMENT**: All templates listed here must be grounded in specific ClrSlate platform specifications.

## Complexity Levels

### Basic
Single ClrSlate component templates based on individual platform specifications.

- **[basic-activity.yaml](../activities/basic-activity.yaml)** - Console activity template
  - **Components**: Activity with console handler
  - **Source**: `activity-specification.md`, `invocationHandlers/console.md`
  - **Configuration**: Simple input/output mapping

### Intermediate  
Multi-component templates that combine 2-3 ClrSlate components based on relationships documented in ClrSlate platform.

- **[tekton-activity.yaml](../activities/tekton-activity.yaml)** - Tekton pipeline activity template
  - **Components**: Activity + PipelineRef reference + Secret handling
  - **Source**: `activity-specification.md`, `invocationHandlers/tekton-pipelineref.md`
  - **Configuration**: Input validation, parameter mapping, secret mounting

- **[basic-pipelineref.yaml](../execution/basic-pipelineref.yaml)** - PipelineRef template
  - **Components**: PipelineRef + Secret mounting + File templating
  - **Source**: `invocationHandlers/tekton-pipelineref.md`, `models.cs`
  - **Configuration**: Schema definition, pipeline mapping, secret configuration

### Advanced
Complex scenarios involving multiple ClrSlate components with intricate relationships based on comprehensive patterns from ClrSlate platform.

*No templates currently available - awaiting development based on ClrSlate platform analysis*

## Development Guidelines

Complexity levels should be determined by:
1. Number of ClrSlate components involved (from platform documentation)
2. Complexity of relationships between components (as documented in ClrSlate platform)
3. Number of configuration options required (based on ClrSlate platform specifications)

Avoid creating artificial complexity levels not supported by the actual ClrSlate platform capabilities.
