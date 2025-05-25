# ClrSlate Catalog Documentation

This directory contains package-specific documentation for the ClrSlate catalog.

## Directory Structure

```
docs/
├── README.md                               # This file
├── metricsServer/                          # MetricsServer package documentation
│   ├── package-design.md                  # Design specifications and architecture
│   └── implementation-summary.md          # Implementation details and status
└── [future-packages]/                     # Documentation for other packages
    ├── package-design.md
    └── implementation-summary.md
```

## Documentation Standards

### For Each Package
- **package-design.md**: Initial design specifications, architecture decisions, and planning
- **implementation-summary.md**: Final implementation details, features, and status

### Document Lifecycle
1. **Design Phase**: Create `package-design.md` with specifications and planning
2. **Implementation Phase**: Update design document as needed
3. **Completion Phase**: Create `implementation-summary.md` with final status
4. **Maintenance Phase**: Update implementation summary for major changes

## Usage Guidelines

- Package-specific documentation goes in `docs/[packageName]/`
- Cross-cutting documentation remains in `.ruru/` for roocode concerns
- All documentation should follow standard markdown formatting
- Include clear examples and usage patterns
- Maintain version history for significant changes