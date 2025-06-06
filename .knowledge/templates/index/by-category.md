# Templates by Category

This index will organize ClrSlate templates by their primary category once templates are created based on the actual ClrSlate platform documentation.

**REQUIREMENT**: All templates listed here must be grounded in specific ClrSlate platform specifications.

## Models (categories/models/)

Templates for defining ClrSlate model structures based on `ModelDefinition.md` and `models.cs`.

*No templates currently available - awaiting development based on ClrSlate platform analysis*

## Activities (categories/activities/)

Templates for activity specifications based on `activity-specification.md`.

### Console Activities
- **[basic-activity.yaml](../activities/basic-activity.yaml)** - Simple console output activity
  - **Source**: `activity-specification.md`, `invocationHandlers/console.md`
  - **Handler Type**: console
  - **Use Case**: Debugging, validation, status display

### Tekton Activities  
- **[tekton-activity.yaml](../activities/tekton-activity.yaml)** - Tekton pipeline execution activity
  - **Source**: `activity-specification.md`, `invocationHandlers/tekton-pipelineref.md`
  - **Handler Type**: tekton.pipelineRef
  - **Use Case**: Complex execution, credential handling

## Execution (execution/)

Templates for execution components based on handler specifications.

### PipelineRef Templates
- **[basic-pipelineref.yaml](../execution/basic-pipelineref.yaml)** - Tekton PipelineRef with secret mounting
  - **Source**: `invocationHandlers/tekton-pipelineref.md`, `models.cs`
  - **Features**: Secret mounting, parameter mapping, file templating

## Secrets (categories/secrets/)

Templates for secret definitions based on `SecretDefinition.md`.

*No templates currently available - awaiting development based on ClrSlate platform analysis*

## Handlers (categories/handlers/)

Templates for invocation handlers based on `invocationHandlers/` directory.

### Known Handler Types (from ClrSlate platform):
- **console** - Based on `invocationHandlers/console.md`
- **tekton-pipelineref** - Based on `invocationHandlers/tekton-pipelineref.md`

*Templates to be developed based on actual handler documentation*

## Wrappers (categories/wrappers/)

Templates for wrapper models based on `WrapperModel.md` and `resource-wrapper-schema.json`.

*No templates currently available - awaiting development based on ClrSlate platform analysis*

## Development Notes

This index will be populated as templates are created based on:
1. Analysis of existing ClrSlate platform documentation
2. Patterns found in `Samples/model-definitions/`
3. Actual ClrSlate component specifications

No speculative or invented templates should be added to this index.
