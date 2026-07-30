---
trigger: always_on
description: Guidelines for writing effective user-oriented documentation for NAB AL Tools
---


# Writing User-Oriented Documentation for NAB AL Tools

Guidelines for creating clear, helpful, and maintainable user documentation for NAB AL Tools extension users.

## Core Principles

### 1. User-First Approach

- Write for Business Central developers and consultants
- Assume familiarity with AL language and Business Central concepts
- Don't assume knowledge of NAB AL Tools-specific features
- **Show, don't just tell** - Use examples and visuals
- Focus on solving real problems, not just listing features

### 2. Maintain Consistency

- Follow the tone and style of existing documentation (README.md, CHANGELOG.md)
- Use consistent terminology throughout all documentation
- Keep formatting patterns uniform across all docs
- Reference related features with proper links

### 3. Keep Documentation Current

- Update docs when features change
- Remove or mark deprecated features clearly
- Include version information for version-specific features
- Review and update examples to match current API

### 4. Balance Detail with Brevity

- Be thorough but not overwhelming
- Avoid overusing marketing terms like "comprehensive", "powerful", "robust"
- Use progressive disclosure (overview → details → advanced)
- Link to related documentation instead of repeating content
- Break complex topics into digestible sections

---

## Documentation Structure

### File Organization

User documentation should be organized in the `docs/` folder with this structure:

```
docs/
├── features/              # Feature-specific documentation
│   ├── xliff-tools.md    # XLIFF translation management
│   ├── language-model.md # AI/LLM integration
│   ├── documentation.md  # Auto-documentation features
│   └── ...
├── guides/                # Step-by-step tutorials
│   ├── getting-started.md
│   ├── translation-workflow.md
│   └── ...
├── reference/             # Technical reference material
│   ├── settings.md       # All extension settings
│   ├── commands.md       # All commands
│   └── ...
└── troubleshooting/       # Common issues and solutions
    └── ...
```

### Document Types

#### Feature Documentation

**Purpose:** Explain what a feature does, why it exists, and how to use it

**Structure:**
```markdown
# Feature Name

Brief 1-2 sentence description of what the feature does.

## Overview

2-3 paragraphs explaining:
- The problem this feature solves
- When to use it
- Key capabilities

## How to Use

Step-by-step instructions with:
- Prerequisites (if any)
- Clear numbered steps
- Expected outcomes

## Examples

Real-world usage examples with:
- Context/scenario
- Code or screenshots
- Results

## Related Features

Links to:
- Complementary features
- Related settings
- Relevant guides

## Troubleshooting

Common issues and solutions
```

#### Guide Documentation

**Purpose:** Walk users through complete workflows or complex tasks

**Structure:**
```markdown
# Guide Title

What this guide helps you accomplish.

## Prerequisites

- Required knowledge
- Required setup
- Required extensions/settings

## Step-by-Step Instructions

### Step 1: [Action]
Detailed instructions...

### Step 2: [Action]
Detailed instructions...

## Best Practices

Tips for optimal results

## Common Pitfalls

What to avoid and why

## Next Steps

What to do after completing this guide
```

#### Reference Documentation

**Purpose:** Provide comprehensive technical details

**Structure:**
```markdown
# Reference Title

Brief description.

## [Item Name]

**Type:** [Setting/Command/API]
**Default:** [Value]
**Since:** v[Version]

Description of what it does.

**Example:**
```json or code example```

**Related:**
- Links to related items
```

---

## Writing Style Guidelines

### Tone and Voice

- **Professional but friendly** - Speak directly to developers
- **Active voice** - "Click the button" not "The button should be clicked"
- **Imperative mood for instructions** - "Run the command" not "You should run"
- **Present tense** - "This feature provides" not "This feature will provide"
- Avoid jargon unless it's standard Business Central terminology

### Terminology

**Use Consistently:**

- **XLF file** or **XLIFF file** (not "translation file" alone)
- **g.xlf file** (not "generated xlf" or "base xlf")
- **trans-unit** or **translation unit** (not "translation entry")
- **AL code** or **AL language** (not just "code")
- **Language Model Tools** (capitalized, for the VS Code API feature)
- **MCP server** (not "MCP Server")
- **Base App** or **BaseApp** (Microsoft's base application)

**Avoid:**

- "Simply" or "just" (what's simple to one person may not be to another)
- "Obviously" or "clearly" (don't assume)
- "Easy" or "trivial" (subjective)
- Unexplained acronyms on first use

### Formatting Conventions

#### Code and Commands

- **Inline code:** Use backticks for: `` `settings` ``, `` `commands` ``, `` `filenames.xlf` ``, `` `code snippets` ``
- **Code blocks:** Use fenced code blocks with language identifiers

```al
// AL code example
procedure MyProcedure()
begin
    Message('Hello World');
end;
```

```json
// JSON configuration
{
    "NAB.SettingName": true
}
```

#### UI Elements

- **Commands:** Use the exact command name with backticks and bold: **`NAB: Refresh XLF files from g.xlf`**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jwikman/nab-al-tools](https://github.com/jwikman/nab-al-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
