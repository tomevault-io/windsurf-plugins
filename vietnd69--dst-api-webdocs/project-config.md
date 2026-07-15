---
trigger: always_on
description: This rule defines the complete workflow for creating, updating, and maintaining index.md files within the DST core-systems documentation structure. It integrates format standards, templates, and quality assurance procedures.
---

# DST Core Systems Index Workflow

This rule defines the complete workflow for creating, updating, and maintaining index.md files within the DST core-systems documentation structure. It integrates format standards, templates, and quality assurance procedures.

## Workflow Overview

The core-systems index workflow consists of four main phases:

1. **Analysis Phase**: Understand the system category and its components
2. **Template Selection**: Choose appropriate format and structure
3. **Content Development**: Create comprehensive documentation content
4. **Quality Assurance**: Verify accuracy and integration

## Phase 1: Analysis Phase

### System Category Assessment

Before creating any index.md file, perform these analysis steps:

#### 1.1 Directory Structure Analysis
- Examine the current directory structure using [list_dir](mdc:dst-api-webdocs/dst-api-webdocs/dst-api-webdocs/dst-api-webdocs/dst-api-webdocs) tool
- Identify all subdirectories and existing documentation files
- Map the relationship between system components
- Determine the hierarchical organization

#### 1.2 Component Inventory
```bash
# Use these commands to understand the system structure
find ./docs/game-scripts/core-systems/[system-category] -name "*.md" -type f
ls -la ./docs/game-scripts/core-systems/[system-category]/
```

#### 1.3 Source Code Verification
- Cross-reference documentation with actual source code in '/dst-scripts' reponsive directory in workspace
- Verify component existence and current functionality
- Check for recent changes in build version 676042
- Identify deprecated or removed components

#### 1.4 Dependency Mapping
- Map dependencies between system categories
- Identify integration points with other core systems
- Document cross-system communication patterns
- Verify API compatibility requirements

### System Classification

Classify the system category using this decision tree:

```
Is this the main core-systems index?
├─ YES → Use Main Index Template
└─ NO → Analyze system purpose
    ├─ Character/Player focused → Character Systems Template
    ├─ Gameplay mechanics → Game Mechanics Template  
    ├─ Developer tools → Development Tools Template
    └─ Technical infrastructure → Infrastructure Template
```

## Phase 2: Template Selection and Customization

### 2.1 Template Selection

Choose the appropriate template from [DST Core Systems Index Templates](mdc:dst-api-webdocs/dst-api-webdocs/dst-api-webdocs/dst-api-webdocs/dst-api-webdocs/.cursor/rules/dst-core-systems-index-templates.mdc):

| System Type | Template | Key Characteristics |
|-------------|----------|-------------------|
| Main Overview | Main Index Template | Top-level system organization |
| Character Systems | Character Systems Template | Player/character functionality |
| Game Mechanics | Game Mechanics Template | Gameplay features |
| Development Tools | Development Tools Template | Developer utilities |
| Infrastructure | Infrastructure Template | Low-level technical systems |

### 2.2 Template Customization

#### Front Matter Customization
```markdown
---
id: [system-category]-overview
title: [System Category Name] Overview
description: Overview of [specific system functionality] in DST API
sidebar_position: [determined by system hierarchy]

last_updated: [current date]
build_version: 676042
change_status: stable
category_type: [system-type]
system_scope: [brief functional scope]
---
```

#### Content Section Mapping
1. **System Purpose**: Define the specific role of this system category
2. **Architecture Overview**: Document the technical organization
3. **Module Inventory**: List all components with current status
4. **Integration Patterns**: Show how systems connect
5. **Development Guidelines**: Provide implementation guidance

### 2.3 Format Compliance

Ensure compliance with [DST Core Systems Index Format](mdc:dst-api-webdocs/dst-api-webdocs/dst-api-webdocs/dst-api-webdocs/dst-api-webdocs/.cursor/rules/dst-core-systems-index-format.mdc):

- Use consistent heading hierarchy (H1 > H2 > H3 > H4)
- Include required sections in specified order
- Apply proper markdown formatting for tables and code blocks
- Maintain cross-reference link consistency
- Follow established terminology conventions

## Phase 3: Content Development

### 3.1 Content Research and Verification

#### Source Code Analysis
Use these commands to gather accurate information:

```bash
# Find relevant source files
grep -r "component_name" scripts/ --include="*.lua"
find scripts/ -name "*[system-name]*" -type f
grep -r "function.*[SystemName]" scripts/ --include="*.lua"
```

#### Module Documentation Verification
- Verify each listed module exists in the source code
- Check component status against actual implementation
- Validate code examples against current build (676042)
- Confirm integration patterns with source analysis

#### Cross-Reference Validation
- Verify all internal links point to existing documentation
- Check that cross-system references are accurate
- Validate external links and dependencies
- Ensure consistent terminology across documentation

### 3.2 Content Writing Guidelines


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vietnd69/dst-api-webdocs](https://github.com/vietnd69/dst-api-webdocs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
