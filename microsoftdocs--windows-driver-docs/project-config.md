---
trigger: always_on
description: This repository contains the source files for the Windows Driver Kit (WDK) conceptual and guidance documentation. The content is published to [learn.microsoft.com/windows-hardware/drivers](/windows-hardware/drivers).
---

# Windows Driver Documentation - Copilot Instructions

## Repository Overview

This repository contains the source files for the Windows Driver Kit (WDK) conceptual and guidance documentation. The content is published to [learn.microsoft.com/windows-hardware/drivers](/windows-hardware/drivers).

### Purpose
- Provides comprehensive conceptual documentation and guidance for Windows device driver development
- Covers driver architecture, design patterns, programming techniques, and best practices
- Serves as the authoritative source for driver development concepts across all driver technologies
- Complements the DDI reference documentation found in the [windows-driver-docs-ddi repository](https://github.com/MicrosoftDocs/windows-driver-docs-ddi)

## Repository Structure

```
windows-driver-docs-pr/
├── {technology}/              # Technology area directories
│   ├── *.md                   # Conceptual documentation files
│   ├── toc.yml                # Table of contents for the technology
│   ├── index.md               # Technology area overview/landing page
│   └── images/                # Images specific to this technology
├── docfx.json                 # Build configuration
├── toc.yml                    # Root table of contents
├── index.yml                  # Root landing page
├── breadcrumbs/               # Breadcrumb navigation
└── includes/                  # Reusable content snippets
```

### Folder Organization
Technology area directories represent major driver categories and development topics:
- **Audio** (`audio/`) - Audio driver concepts and design patterns
- **Display** (`display/`) - Display and graphics driver guidance
- **Kernel** (`kernel/`) - Kernel-mode driver fundamentals and techniques
- **Network** (`netcx/`, `network/`) - Network driver development
- **Storage** (`storage/`) - Storage driver concepts
- **USB** (`usbcon/`) - USB driver development
- **WDF** (`wdf/`) - Windows Driver Framework concepts
- **Install** (`install/`) - Driver installation and INF files
- **Debugger** (`debugger/`, `debuggercmds/`) - Debugging documentation
- And many other specialized driver areas

Each technology directory typically contains conceptual articles, how-to guides, design patterns, and technology-specific guidance.

## File Naming Conventions

Unlike the DDI reference documentation, conceptual documentation files use descriptive, topic-based naming:

- Use lowercase with hyphens for multi-word names: `defining-i-o-control-codes.md`
- Use descriptive names that reflect the content: `introduction-to-wdm.md`, `writing-dpc-routines.md`
- Landing pages are typically named `index.md`
- Table of contents files are named `toc.yml`

## Content Structure

Each documentation file contains:

### YAML Frontmatter (Required)
Every file must start with YAML frontmatter containing metadata:

```yaml
---
title: {Article title}
description: {Brief description of the article}
keywords: ["{keyword1}", "{keyword2}", "{keyword3}"]
ms.date: {Date in MM/DD/YYYY format}
ms.topic: {concept-article|design-pattern|overview|reference|how-to}
---
```

#### Required AI Usage Metadata

**IMPORTANT**: When Copilot creates or modifies content, ensure that `ai-usage: ai-assisted` is included as a metadata attribute in the YAML frontmatter. If this attribute is not present, add it.

### Common Metadata Patterns

#### Topic Types (ms.topic):
- `concept-article` - Explanatory/conceptual content
- `design-pattern` - Design patterns and architectural guidance
- `overview` - High-level overviews and introductions
- `reference` - Reference information (not API reference, but tables, lists, etc.)
- `how-to` - Step-by-step procedures

#### Other Common Metadata:
- `targetos: Windows` - Specifies target operating system
- `ms.custom` - Custom tags for special handling
- `ms.update-cycle` - Update frequency for content that changes regularly

### Content Sections

Conceptual documentation is more flexible than API reference, but should follow these general patterns:

#### Standard Structure:
1. **Title** - H1 heading matching the title in frontmatter
2. **Introduction** - Brief overview of the topic and its purpose
3. **Main content** - Organized with H2 and H3 headings as appropriate
4. **Code examples** - When applicable, showing practical implementations
5. **Related topics** - Links to related articles

#### Common Section Types:
- Overview and introduction sections
- Architecture and design explanations
- Step-by-step procedures and how-tos
- Best practices and guidelines
- Examples and sample code
- Troubleshooting information
- Related topics and see-also sections

## Link Formatting Guidelines

### Site-Relative Links (Required)
**IMPORTANT**: Use site-relative links instead of fully qualified URLs for learn.microsoft.com content.

✅ **Correct:**
```markdown
[Windows Hardware Developer](/windows-hardware)
[Driver Development](/windows-hardware/drivers)
[API Reference](/windows-hardware/drivers/ddi)
```

❌ **Incorrect:**
```markdown  
[Windows Hardware Developer](https://learn.microsoft.com/windows-hardware)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MicrosoftDocs/windows-driver-docs](https://github.com/MicrosoftDocs/windows-driver-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
