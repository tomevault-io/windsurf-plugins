---
trigger: always_on
description: Cursor rules for documentation creation, maintenance, and validation
---


# Documentation Work and Maintenance

## Documentation Context

When working on documentation, include these essential sources:

### Required Documentation

- [ABOUT.documentation.md](mdc:documentation/dev/ABOUT.documentation.md) - Documentation standards and system
- [README.documentation-testing.md](mdc:documentation/dev/test/README.documentation-testing.md) - Documentation validation system
- [README.documentation-index.md](mdc:documentation/README.documentation-index.md) - Complete documentation index

### Template System

- [TEMPLATE.documentation-standard.md](mdc:documentation/dev/templates/TEMPLATE.documentation-standard.md) - Standard documentation template
- [TEMPLATE.guide.md](mdc:documentation/dev/templates/TEMPLATE.guide.md) - Step-by-step guide template
- [TEMPLATE.troubleshooting.md](mdc:documentation/dev/templates/TEMPLATE.troubleshooting.md) - Troubleshooting template
- [TEMPLATE.custom.md](mdc:documentation/dev/templates/TEMPLATE.custom.md) - Custom document template

## Documentation Standards

### File Naming Conventions

| Prefix             | Purpose                           | Example                              |
| ------------------ | --------------------------------- | ------------------------------------ |
| `ABOUT.`           | Meta-documents explaining systems | `ABOUT.documentation.md`             |
| `README.`          | Main documentation files          | `README.container-testing.md`        |
| `TEMPLATE.`        | Template files for consistency    | `TEMPLATE.documentation-standard.md` |
| `GUIDE.`           | Step-by-step guides               | `GUIDE.docker-setup.md`              |
| `TROUBLESHOOTING.` | Problem-solving docs              | `TROUBLESHOOTING.build-issues.md`    |

### YAML Front Matter Requirements

Every documentation file must include complete YAML front matter:

```yaml
---
title: "Document Title"
description: "Brief purpose description"
template: "TEMPLATE.documentation-standard"
version: "1.0.0"
last_updated: "2025-01-03"
author: "Author Name"
tags: ["tag1", "tag2", "tag3"]
categories: ["category1", "category2"]
difficulty: "intermediate" # beginner, intermediate, advanced
prerequisites: ["prereq1", "prereq2"]
related_docs:
  - "related-doc1.md"
  - "related-doc2.md"
dependencies:
  - "required-file1"
  - "required-directory/"
llm_context: "high" # high, medium, low
search_keywords: ["keyword1", "keyword2", "keyword3"]
---
```

### Template Compliance

- **Use appropriate templates** for different document types
- **Include all required sections** from the template
- **Follow template structure** for consistency
- **Custom documents** can use `TEMPLATE.custom` for flexibility

## Documentation Workflow

### Before Creating Documentation

1. **Check existing documentation**: Review `documentation/README.documentation-index.md`
2. **Choose appropriate template**: Select from `documentation/dev/templates/`
3. **Plan document relationships**: Identify related documents for `related_docs`

### During Documentation Creation

1. **Start with YAML front matter**: Complete all required fields
2. **Follow template structure**: Use the selected template as a guide
3. **Include comprehensive content**: Purpose, When, How, What, Troubleshooting
4. **Add LLM context section**: For AI optimization
5. **Use proper formatting**: Markdown best practices

### After Creating Documentation

1. **Run documentation validation**: `cd testing && make lint-docs`
2. **Check index completeness**: `cd testing && make lint-index`
3. **Update documentation index**: Add new file to `README.documentation-index.md`
4. **Test template compliance**: Verify all required sections are present

## Documentation Validation

### Quick Validation

```bash
cd testing
make lint-docs-quick
```

### Full Validation

```bash
cd testing
make lint-docs
```

### Index Validation

```bash
cd testing
make lint-index
```

### Validation Checks

- **YAML front matter completeness**: All required fields present
- **Template compliance**: Document follows selected template
- **Index completeness**: All files referenced in documentation index
- **Link validation**: Internal links work correctly
- **Metadata validity**: Field values match valid options

## Documentation Categories

### Development Documentation

- **Purpose**: Development setup, workflows, standards
- **Location**: `documentation/dev/`
- **Templates**: `TEMPLATE.documentation-standard`, `TEMPLATE.guide`
- **Examples**: `README.development.md`, `README.container-testing.md`

### Architecture Documentation

- **Purpose**: System design, component relationships
- **Location**: `documentation/dev/architecture/`
- **Templates**: `TEMPLATE.architecture`, `TEMPLATE.documentation-standard`
- **Examples**: `README.architecture.md`

### Template Documentation

- **Purpose**: Document templates and standards
- **Location**: `documentation/dev/templates/`
- **Templates**: `TEMPLATE.template`
- **Examples**: `TEMPLATE.documentation-standard.md`

### Testing Documentation

- **Purpose**: Testing systems and validation processes
- **Location**: `documentation/dev/test/`
- **Templates**: `TEMPLATE.documentation-standard`
- **Examples**: `README.container-testing.md`, `README.documentation-testing.md`

## LLM Optimization

### Context Levels


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SiriusScan/Sirius](https://github.com/SiriusScan/Sirius) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
