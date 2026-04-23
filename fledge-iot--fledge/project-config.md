---
trigger: always_on
description: description: "Cursor AI rules for Fledge documentation - covers reStructuredText, Sphinx, file naming, and content guidelines"
---


---
description: "Cursor AI rules for Fledge documentation - covers reStructuredText, Sphinx, file naming, and content guidelines"
globs: 
  - "docs/**/*"
  - "*.rst"
alwaysApply: false
author: "Ashish Jabble"
---

# Documentation Directory Cursor Rules

## Overview
This file contains specific rules for working with documentation in the `/docs` directory of the Fledge project. These rules supplement the main project cursor rules and focus on documentation-specific patterns and conventions.

## Documentation Framework
- **Format**: reStructuredText (.rst) format exclusively
- **Build System**: Sphinx documentation generator
- **Theme**: sphinx_rtd_theme (Read the Docs theme)
- **Configuration**: All settings in `docs/conf.py`

## 🚫 "Fledge" Branding Guidelines - MINIMIZE USAGE

**CRITICAL RULE: Avoid "Fledge" in naming wherever possible**

### What to Avoid:
- ❌ Image files: `fledge_architecture.png`
- ❌ Directory names: `fledge_authentication/`
- ❌ File names: `fledge_configuration.rst`
- ❌ Headings: "Fledge Authentication Setup"
- ❌ Repetitive content: "Fledge does this... Fledge provides that..."

### What to Use Instead:
- ✅ Image files: `architecture_overview.png`, `auth_flow.png`
- ✅ Directory names: `authentication/`, `configuration/`, `monitoring/`
- ✅ File names: `authentication.rst`, `configuration.rst`
- ✅ Headings: "Authentication Setup", "Configuration Guide"
- ✅ Content alternatives: "the platform", "the system", "this feature"

### When "Fledge" IS Appropriate:
- Main title pages and introductory content
- External references and comparisons
- Installation package names
- API endpoint references where it's part of the actual name

## File Organization

### Directory Structure
- `/docs/` - Main documentation root
- `/docs/_static/` - Static assets (CSS, images that aren't content)
- `/docs/_templates/` - Custom Sphinx templates
- `/docs/images/` - Documentation images and screenshots
- `/docs/quick_start/` - Getting started guides
- `/docs/plugin_developers_guide/` - Plugin development documentation
- `/docs/rest_api_guide/` - REST API documentation
- `/docs/building_fledge/` - Build and installation guides
- `/docs/monitoring/` - System monitoring documentation
- `/docs/fledge-rule-DataAvailability/` - Built-in Data Availability rule plugin docs
- `/docs/fledge-rule-Threshold/` - Built-in Threshold rule plugin docs
- `/docs/fledge-north-OMF.rst` - Built-in OMF north plugin documentation
- `/docs/keywords/` - Plugin categorization keywords and mappings
- `/docs/fledge_plugins.rst` - Master plugin list with conditional hyperlinks

**DIRECTORY NAMING GUIDELINES:**
- **AVOID "fledge" in new directory names** - use functional descriptions
- Use topic-based naming: `authentication/` instead of `fledge_authentication/`
- Keep directory names lowercase with underscores
- Focus on the purpose/feature rather than product branding

### File Naming Conventions
- Use lowercase with underscores: `file_name.rst`
- Index files: `index.rst` for each directory
- Numbered files for version/download info: `91_version_history.rst`, `92_downloads.rst`
- Descriptive names reflecting content: `securing.rst`, `troubleshooting_pi_server_integration.rst`
- **AVOID "fledge" in filenames** - use functional descriptions: `authentication.rst` instead of `fledge_authentication.rst`
- Focus on the topic/feature being documented

## reStructuredText Style Guidelines

### Heading Hierarchy
Follow this exact hierarchy for consistency:
```rst
***************
Document Title (Level 1)
***************

===============
Major Section (Level 2)
===============

Minor Section (Level 3)
-----------------------

Subsection (Level 4)
^^^^^^^^^^^^^^^^^^^^

Sub-subsection (Level 5)
"""""""""""""""""""""""""
```

**IMPORTANT NAMING CONVENTIONS:**
- **AVOID "Fledge" in headings** unless absolutely necessary for context
- Use descriptive, functional titles: "Authentication Configuration" instead of "Fledge Authentication Configuration"
- Focus on the feature/functionality rather than the product name
- Keep headings concise and user-focused

### Document Structure
1. **Title**: Use level 1 heading with asterisks above and below
2. **Introduction**: Brief overview of the document's purpose
3. **Table of Contents**: Use `.. toctree::` for sections with multiple pages
4. **Main Content**: Organized with appropriate heading levels
5. **Cross-references**: Link to related documentation

### Code Blocks
```rst
.. code-block:: language
   :linenos:
   :emphasize-lines: 2,3

   code here
```

### Common Directives
- `.. note::` - Important information
- `.. warning::` - Critical warnings
- `.. code-block::` - Code examples
- `.. image::` - Images with proper alt text
- `.. toctree::` - Table of contents trees

### Images and Media
- Store images in `/docs/images/` directory
- Use descriptive filenames: `architecture_overview.png` (avoid "fledge_" prefix)
- **AVOID "Fledge" in image filenames** - use descriptive terms like `architecture_overview.png` instead of `fledge_architecture_overview.png`
- Always include alt text: `.. image:: images/filename.png :alt: Description`
- Optimize images for web (reasonable file sizes)
- Use subdirectories in images/ for organization by topic

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fledge-iot) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
