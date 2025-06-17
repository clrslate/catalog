# Application Scaffolding

## Overview

Building blocks for scaffolding applications using popular tools and templates. This package provides a unified set of activities for generating new projects across multiple technologies and frameworks using a single package metadata and icon.

## Structure

This package uses a unified structure with:

- **Single metadata.yaml** with `Generic.Scaffolding` icon and `#4CAF50` color
- **All activities directly in activities/** folder
- **Consistent branding** across all scaffolding tools

## Activities

### Template-Based Generation

#### Generate Project with Cookiecutter

**Activity**: `scaffolding.cookiecutter.generate`

Scaffold a new project using a Cookiecutter template.

**Parameters**:

- `templateUrl` (required): URL of the Cookiecutter template repository
- `outputDir` (required): Directory to generate the project in
- `noInput` (optional): Skip prompts and use default values (true/false, default: false)

### CLI-Based Project Creation

#### Create .NET Project with dotnet new

**Activity**: `scaffolding.dotnet.new`

Scaffold a new .NET project using the dotnet CLI.

**Parameters**:

- `template` (required): Name of the .NET template to use (e.g., console, webapi)
- `output` (required): Output directory for the project

#### Create React App with NPX

**Activity**: `scaffolding.npx.create-react-app`

Scaffold a new React application using NPX.

**Parameters**:

- `appName` (required): Name of the new React application
- `template` (optional): React template to use (e.g., typescript)

#### Create Next.js App with NPX

**Activity**: `scaffolding.npx.create-next-app`

Scaffold a new Next.js application using NPX.

**Parameters**:

- `appName` (required): Name of the new Next.js application
- `typescript` (optional): Use TypeScript template (true/false, default: false)

#### Generate Maven Project with Archetype

**Activity**: `scaffolding.maven.archetype.generate`

Scaffold a new Java project using Maven Archetypes.

**Parameters**:

- `archetypeGroupId` (required): Group ID of the Maven Archetype
- `archetypeArtifactId` (required): Artifact ID of the Maven Archetype
- `groupId` (required): Group ID for the generated project
- `artifactId` (required): Artifact ID for the generated project

#### Generate Project with Yeoman

**Activity**: `scaffolding.yo.generator`

Scaffold a new project using a Yeoman generator.

**Parameters**:

- `generator` (required): Name of the Yeoman generator to use
- `outputDir` (required): Directory to generate the project in

#### Create Flask App

**Activity**: `scaffolding.flask.create-app`

Scaffold a new Flask application using Flask CLI.

**Parameters**:

- `appName` (required): Name of the new Flask application
- `directory` (required): Directory to generate the Flask application in

#### Create Angular App with Angular CLI

**Activity**: `scaffolding.angular.cli.new`

Scaffold a new Angular application using Angular CLI.

**Parameters**:

- `appName` (required): Name of the new Angular application
- `style` (optional): CSS preprocessor to use (e.g., SCSS, CSS)

## Usage Examples

### Template-Based Generation

```yaml
# Generate Django project from Cookiecutter
activity: scaffolding.cookiecutter.generate
inputs:
  templateUrl: "https://github.com/cookiecutter/cookiecutter-django"
  outputDir: "./my-django-project"
  noInput: "false"
```

### CLI-Based Project Creation

```yaml
# Create .NET Web API
activity: scaffolding.dotnet.new
inputs:
  template: "webapi"
  output: "./my-web-api"

# Create React application with TypeScript
activity: scaffolding.npx.create-react-app
inputs:
  appName: "my-react-app"
  template: "typescript"

# Create Next.js application with TypeScript
activity: scaffolding.npx.create-next-app
inputs:
  appName: "my-nextjs-app"
  typescript: "true"

# Generate Maven quickstart project
activity: scaffolding.maven.archetype.generate
inputs:
  archetypeGroupId: "org.apache.maven.archetypes"
  archetypeArtifactId: "maven-archetype-quickstart"
  groupId: "com.example"
  artifactId: "my-java-app"

# Generate project with Yeoman
activity: scaffolding.yo.generator
inputs:
  generator: "webapp"
  outputDir: "./my-webapp"

# Create Flask application
activity: scaffolding.flask.create-app
inputs:
  appName: "my-flask-app"
  directory: "./flask-projects"

# Create Angular application with SCSS
activity: scaffolding.angular.cli.new
inputs:
  appName: "my-angular-app"
  style: "scss"
```

### Multi-Technology Workflow

```yaml
# Full-stack development setup
steps:
  - name: Create Backend API
    activity: scaffolding.dotnet.new
    inputs:
      template: "webapi"
      output: "./backend"

  - name: Create Frontend App
    activity: scaffolding.angular.cli.new
    inputs:
      appName: "frontend"
      style: "scss"

  - name: Create Mobile App
    activity: scaffolding.npx.create-react-app
    inputs:
      appName: "mobile-app"
      template: "typescript"

  - name: Create Documentation
    activity: scaffolding.cookiecutter.generate
    inputs:
      templateUrl: "https://github.com/audreyfeldroy/cookiecutter-pypackage"
      outputDir: "./docs"
```

## Tool Requirements

Different activities require different tools to be installed:

- **Cookiecutter**: `pip install cookiecutter`
- **NPX**: Comes with npm/Node.js
- **.NET CLI**: Download from https://dotnet.microsoft.com/download
- **Maven**: Download from https://maven.apache.org/
- **Yeoman**: `npm install -g yo`
- **Flask**: `pip install flask`
- **Angular CLI**: `npm install -g @angular/cli`

## Directory Structure

```
catalog/application-scaffolding/
├── metadata.yaml                    # Unified package metadata
├── activities/                      # All activities in single directory
│   ├── cookiecutter-generate.yaml
│   ├── dotnet-new.yaml
│   ├── npx-create-react-app.yaml
│   ├── npx-create-next-app.yaml
│   ├── maven-archetype-generate.yaml
│   ├── yeoman-generator.yaml
│   ├── flask-create-app.yaml
│   └── angular-cli-new.yaml
└── README.md                       # This documentation
```

## Best Practices

1. **Consistent Naming**: Use kebab-case for project names
2. **Directory Organization**: Structure projects by technology or domain
3. **Template Selection**: Choose appropriate templates for project requirements
4. **Version Control**: Initialize git repositories for new projects
5. **Environment Setup**: Ensure required CLI tools are installed

## Notes

- All activities use the unified `Generic.Scaffolding` icon and `#4CAF50` color
- Each activity maintains its individual functionality while sharing common branding
- Console handlers provide structure and validation for scaffolding operations
- For production use, consider implementing corresponding PipelineRef handlers
- Individual technology icons are intentionally ignored for unified presentation
