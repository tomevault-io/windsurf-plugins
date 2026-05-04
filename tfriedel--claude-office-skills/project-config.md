---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a skills repository for Office document manipulation (PPTX, DOCX, XLSX, PDF). Each skill provides workflows, scripts, and documentation for working with specific file formats. The repository follows a pattern established in `skills-system.md` which defines mandatory workflows for document creation and editing.

## Repository Structure

```
public/
├── pptx/           # PowerPoint presentation skills
│   ├── SKILL.md    # Main workflow documentation
│   ├── ooxml.md    # OOXML editing guide
│   ├── html2pptx.md # HTML-to-PPTX conversion guide
│   ├── scripts/    # Python and JS utilities
│   └── ooxml/      # OOXML validation and schemas
├── docx/           # Word document skills
│   ├── SKILL.md    # Main workflow documentation
│   ├── ooxml.md    # OOXML editing guide
│   ├── docx-js.md  # JavaScript library documentation
│   └── scripts/    # Python utilities
├── pdf/            # PDF manipulation skills
│   ├── SKILL.md    # Main workflow documentation
│   ├── REFERENCE.md # Advanced features and examples
│   └── FORMS.md    # PDF form filling guide
└── xlsx/           # Excel spreadsheet skills
    └── SKILL.md    # Main workflow documentation

outputs/            # All skill-generated documents (gitignored)
└── <document-name>/ # One directory per document project
    ├── *.pptx      # Final outputs
    ├── *.docx      # Final outputs
    ├── *.pdf       # Final outputs
    ├── *.xlsx      # Final outputs
    ├── unpacked/   # Unpacked OOXML files
    ├── *.json      # Inventories and replacements
    ├── *.html      # HTML slides
    └── images/     # Generated images
```

## Key Architecture Principles

### Skills-Based System
The repository follows a mandatory skills-check system (defined in `skills-system.md`):
1. **Before writing ANY code**: Check if a skill exists for the task
2. **If YES**: Read the corresponding SKILL.md and follow it exactly
3. **If NO**: Only then proceed with custom code

This prevents reinventing workflows that already exist in the skills documentation.

### Two-Phase Approach for Complex Operations
Most OOXML editing workflows follow this pattern:
1. **Unpack**: Extract the Office file to raw XML using `venv/bin/python ooxml/scripts/unpack.py`
2. **Edit**: Modify XML files directly
3. **Validate**: Check changes using `venv/bin/python ooxml/scripts/validate.py --original <file>`
4. **Pack**: Repackage to Office file using `venv/bin/python ooxml/scripts/pack.py`

### Read-First Policy
All SKILL.md files must be read **completely** before starting work. Never set range limits when reading these files - they contain critical workflow steps and validation requirements.

### Output Directory Convention
**MANDATORY**: All files created or edited using any skill workflow MUST be written to:
```
outputs/<fitting-name-for-document>/
```

Where `<fitting-name-for-document>` is a descriptive, lowercase, hyphenated name for the document being worked on.

**Examples**:
- `outputs/quarterly-sales-report/` - for a sales presentation
- `outputs/employee-handbook/` - for a Word document
- `outputs/budget-2024/` - for an Excel file
- `outputs/project-proposal/` - for a PDF document

**Rules**:
1. Create the outputs directory if it doesn't exist
2. All intermediate files (unpacked XML, JSON inventories, HTML files, images, etc.) go in this directory
3. Final output files (PPTX, DOCX, XLSX, PDF) go in this directory
4. Never write skill-generated files to the repository root or public/ directories
5. Use descriptive names that clearly identify the document's purpose

## Development Setup

### Python Environment
The repository uses a Python virtual environment:

```bash
# Install/update dependencies
venv/bin/pip install -r requirements.txt
```

**CRITICAL**: Always use `venv/bin/python` for all Python commands. NEVER use system Python or assume venv is activated. All Python commands in this document use explicit venv paths.

### Node.js Dependencies
JavaScript tools for html2pptx workflow:

```bash
# Install dependencies (includes playwright chromium browser)
npm install
```

### System Tools
Required system dependencies (should be pre-installed):
- LibreOffice: `soffice` (for PDF conversion)
- Poppler: `pdftoppm` (for PDF to image conversion)
- Pandoc: `pandoc` (for document text extraction)

## Common Commands

### PowerPoint (PPTX)

**Text extraction**:
```bash
venv/bin/python -m markitdown file.pptx
```

**Unpack for XML editing**:
```bash
venv/bin/python public/pptx/ooxml/scripts/unpack.py input.pptx outputs/<document-name>/unpacked/
```

**Validate after editing**:
```bash
venv/bin/python public/pptx/ooxml/scripts/validate.py outputs/<document-name>/unpacked/ --original input.pptx
```

**Repack to PPTX**:
```bash
venv/bin/python public/pptx/ooxml/scripts/pack.py outputs/<document-name>/unpacked/ outputs/<document-name>/final.pptx
```

**Create thumbnail grid for visual analysis**:
```bash
venv/bin/python public/pptx/scripts/thumbnail.py template.pptx outputs/<document-name>/thumbnails [--cols 4]
```

**Rearrange slides (duplicate, reorder, delete)**:
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tfriedel/claude-office-skills](https://github.com/tfriedel/claude-office-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
