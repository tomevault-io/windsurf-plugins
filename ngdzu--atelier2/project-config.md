---
trigger: always_on
description: This document defines the markdown coding standards and best practices for documentation in the LuxeNail salon website project. These guidelines ensure consistent, readable, and maintainable documentation that renders correctly across different markdown processors.
---


# Markdown Guidelines

## Overview

This document defines the markdown coding standards and best practices for documentation in the LuxeNail salon website project. These guidelines ensure consistent, readable, and maintainable documentation that renders correctly across different markdown processors.

**Key Principles:**
- **Valid Syntax** - All markdown must be valid and render correctly
- **Readability First** - Format for human readability, not just rendering
- **Consistency** - Use consistent formatting patterns throughout
- **Accessibility** - Make documents navigable with proper links and structure
- **Maintainability** - Write markdown that's easy to update and maintain

---

## 1. Syntax Validation

### Valid Markdown Syntax

- **Verify headers, lists, code fences, and tables render correctly**
- **Test complex sections** - Preview files (e.g., VS Code Markdown preview) before committing
- **Check for syntax errors** - Ensure all markdown elements are properly closed

### Common Syntax Issues

- **Unclosed code fences**: Every ` ``` ` must have a matching closing ` ``` `
- **Mismatched headers**: Use consistent header levels (don't skip levels)
- **Broken lists**: Ensure proper indentation and list markers
- **Invalid links**: Verify all links resolve correctly

---

## 2. Headers

### Header Levels

- **Use hierarchical structure**: Don't skip header levels (e.g., don't go from `##` to `####`)
- **Consistent formatting**: Use ATX-style headers (`# Header`) not Setext-style (`===`)
- **Descriptive titles**: Headers should clearly describe the section content

```markdown
# ✅ Good: Hierarchical structure
# Main Title
## Section
### Subsection
## Another Section

# ❌ Bad: Skipping levels
# Main Title
#### Subsection (skipped ## and ###)
```

### Header Best Practices

- **One H1 per document**: Use `#` for document title only
- **Use H2 for major sections**: `##` for main sections
- **Use H3+ for subsections**: `###` for subsections within sections
- **Table of contents**: Consider adding TOC for long documents

---

## 3. Code Blocks

### Fenced Code Blocks

- **Always use fenced code blocks** with language identifiers:
  ```markdown
  ```typescript
  // TypeScript code here
  ```
  
  ```tsx
  // React/TSX code here
  ```
  
  ```bash
  # Commands here
  ```
  ```

- **Language identifiers**: Use appropriate language tags (`typescript`, `tsx`, `ts`, `javascript`, `jsx`, `js`, `json`, `yaml`, `yml`, `css`, `scss`, `html`, `bash`, `shell`, `sql`, etc.)

### Code Block Best Practices

- **No comments in shell/bash code blocks** - Comments make it difficult to copy and paste commands
  ```markdown
  # ❌ Bad: Comments in code block
  ```bash
  npm install  # Install dependencies
  npm run dev  # Start dev server
  ```
  
  # ✅ Good: Explanatory text outside code block
  Install dependencies:
  
  ```bash
  npm install
  ```
  
  Then start the development server:
  
  ```bash
  npm run dev
  ```
  ```

- **Ensure code samples compile or type-check**: Verify code examples are correct TypeScript/React code
- **Include context**: Add brief explanation before code blocks when needed
- **Keep code blocks focused**: Show only relevant code, not entire files

### Inline Code

- **Use backticks for inline code**: `` `variableName` ``, `` `function()` ``, `` `Component` ``
- **Use for**: File names, function names, variable names, component names, props, commands, technical terms, API endpoints
- **Don't use for**: Emphasis (use `*italic*` or `**bold**` instead)

---

## 4. Tables

### Table Syntax

- **Ensure each row has the same number of `|` separators**
- **Include alignment row**: Use `| --- |` or `| :--- |` or `| :---: |` or `| ---: |`
- **Leave blank lines** before and after tables for readability

```markdown
# ✅ Good: Properly formatted table
| Column 1 | Column 2 | Column 3 |
|----------|----------|-----------|
| Value 1  | Value 2  | Value 3   |
| Value 4  | Value 5  | Value 6   |

# ❌ Bad: Missing alignment row
| Column 1 | Column 2 |
| Value 1  | Value 2  |

# ❌ Bad: Inconsistent separators
| Column 1 | Column 2 | Column 3 |
| Value 1  | Value 2  |  (missing separator)
```

### Table Best Practices

- **Avoid trailing whitespace** inside table cells
- **Align columns consistently**: Use alignment markers when helpful
- **Keep tables simple**: Complex tables may not render well
- **Use tables for structured data**: Not for layout (use HTML/CSS if needed)

### Table Alignment

```markdown
# Left-aligned (default)
| Column | Column |
|--------|--------|
| Value  | Value  |

# Center-aligned
| Column | Column |
|:------:|:------:|
| Value  | Value  |

# Right-aligned
| Column | Column |
|-------:|------:|
| Value  | Value  |

# Mixed alignment
| Left | Center | Right |
|:-----|:------:|------:|
| A    | B      | C     |
```

---

## 5. Links

### Markdown Link Syntax

- **Always use markdown links for file references** - Makes documents navigable
  ```markdown
  # ✅ Good: Markdown link
  See [BookingFlow Component](../components/BookingFlow.tsx) for the booking implementation.
  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ngdzu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
