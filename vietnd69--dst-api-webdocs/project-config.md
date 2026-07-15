---
trigger: always_on
description: This rule defines the standardized format for index.md files within the core-systems directory structure. These files serve as navigation hubs and overview pages for each major system category.
---

# DST Core Systems Index Format

This rule defines the standardized format for index.md files within the core-systems directory structure. These files serve as navigation hubs and overview pages for each major system category.

## File Purpose

Index.md files in core-systems serve multiple purposes:
- **Navigation Hub**: Provide organized access to all modules within a system category
- **System Overview**: Explain the purpose and scope of the system category
- **Integration Guide**: Show how modules within the category work together
- **Change Tracking**: Document recent changes and updates across the category

## Standard File Structure

### Front Matter Template

All core-systems index.md files must include:

```markdown
---
id: [system-category-id]
title: [System Category Name] Overview
description: Overview of [system category] functionality in DST API
sidebar_position: 0

last_updated: YYYY-MM-DD
build_version: 676042
change_status: stable
category_type: core-system
system_scope: [brief scope description]
---
```

### Document Structure Template

```markdown
# [System Category Name] Overview

## Build Information
Current documentation based on build version: **676042**
Last updated: **YYYY-MM-DD**

## System Purpose

[2-3 paragraph explanation of what this system category provides to DST]

### Key Responsibilities
- [Primary responsibility 1]
- [Primary responsibility 2] 
- [Primary responsibility 3]

### System Scope
[Define what is included and excluded from this system category]

## Architecture Overview

### System Components
[High-level overview of how components in this category work together]

### Data Flow
[Brief description of how data flows through this system]

### Integration Points
[How this system connects with other core systems]

## Recent Changes

| Build | Date | Component | Change Type | Description |
|-------|------|-----------|-------------|-------------|
| 676042 | 2024-XX-XX | [Component A](mdc:dst-api-webdocs/dst-api-webdocs/subdir/component-a.md) | added | Added new functionality X |
| 676042 | 2024-XX-XX | [Component B](mdc:dst-api-webdocs/dst-api-webdocs/subdir/component-b.md) | modified | Changed parameter structure |
| 675312 | 2023-11-15 | [Component C](mdc:dst-api-webdocs/dst-api-webdocs/subdir/component-c.md) | deprecated | Will be removed in build 690000 |

## Core Modules

### [Subcategory 1 Name]
[Brief description of subcategory purpose]

| Module | Status | Description | Key Features |
|--------|--------|-------------|--------------|
| [Module A](mdc:dst-api-webdocs/dst-api-webdocs/subcategory1/module-a.md) | stable | Brief description | Feature 1, Feature 2 |
| [Module B](mdc:dst-api-webdocs/dst-api-webdocs/subcategory1/module-b.md) | added in 676042 | Brief description | Feature 1, Feature 2 |

### [Subcategory 2 Name]
[Brief description of subcategory purpose]

| Module | Status | Description | Key Features |
|--------|--------|-------------|--------------|
| [Module C](mdc:dst-api-webdocs/dst-api-webdocs/subcategory2/module-c.md) | stable | Brief description | Feature 1, Feature 2 |
| [Module D](mdc:dst-api-webdocs/dst-api-webdocs/subcategory2/module-d.md) | modified in 676042 | Brief description | Feature 1, Feature 2 |

## Common Usage Patterns

### Pattern 1: [Pattern Name]
```lua
-- Example showing common usage pattern
local example = SomeModule.CreateInstance()
example:Configure(settings)
example:Initialize()
```

### Pattern 2: [Pattern Name]
```lua
-- Example showing integration pattern
local systemA = CoreSystemA.GetInstance()
local systemB = CoreSystemB.GetInstance()
systemA:ConnectTo(systemB)
```

## System Dependencies

### Required Systems
- [Required System 1]: [Why it's required]
- [Required System 2]: [Why it's required]

### Optional Systems
- [Optional System 1]: [How it enhances functionality]
- [Optional System 2]: [How it enhances functionality]

## Performance Considerations

### Memory Usage
[Guidelines for memory efficiency in this system]

### Performance Optimizations
[Key optimization strategies for this system category]

### Scaling Considerations
[How this system handles increased load or complexity]

## Development Guidelines

### Best Practices
- [Best practice 1 for working with this system]
- [Best practice 2 for working with this system]
- [Best practice 3 for working with this system]

### Common Pitfalls
- [Common mistake 1 and how to avoid it]
- [Common mistake 2 and how to avoid it]

### Testing Strategies
[Recommended approaches for testing components in this system]

## Related Systems

| System | Relationship | Integration Points |
|--------|--------------|-------------------|
| [System A](mdc:dst-api-webdocs/other-category/index.md) | [Type of relationship] | [How they connect] |
| [System B](mdc:dst-api-webdocs/other-category/index.md) | [Type of relationship] | [How they connect] |

## Troubleshooting

### Common Issues
| Issue | Symptoms | Solution |
|-------|----------|----------|
| [Issue 1] | [What users see] | [How to fix] |
| [Issue 2] | [What users see] | [How to fix] |

### Debugging Tips
- [Debugging tip 1]
- [Debugging tip 2]
- [Debugging tip 3]

## Migration Notes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vietnd69/dst-api-webdocs](https://github.com/vietnd69/dst-api-webdocs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
