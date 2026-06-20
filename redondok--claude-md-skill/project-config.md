---
trigger: always_on
description: This skill provides comprehensive guidance for generating GitHub Flavored
---

# GitHub Flavored Markdown (GFM) Skill

**Version:** 1.2.0
**Purpose:** Transform AI markdown generation to be 100% markdownlint-compliant
**Target:** Zero markdownlint violations in all generated markdown

## Overview

This skill provides comprehensive guidance for generating GitHub Flavored
Markdown (GFM) that passes markdownlint validation without errors. Markdown
generated using this skill should be immediately usable in VSCode without
validation warnings. This skill is used whenever markdown is generated, unless
contrary instructions are given.

## Core Principles

### 1. Blank Lines Are Not Optional

**CRITICAL:** Blank lines around block elements are MANDATORY, not stylistic
choices.

Required blank lines:

- Before and after ALL lists (MD032)
- Before and after ALL headings (MD022)
- Before and after ALL code blocks (MD031)
- Between ALL block-level elements

### 2. Consistency Is Required

- Use ONE list marker style throughout (`-` recommended)
- Use ONE heading style throughout (ATX `#` recommended)
- Maintain consistent indentation
- Follow consistent patterns

### 3. Structure Matters

- Heading hierarchy must increment by one (1→2→3, not 1→3)
- Only ONE level 1 heading per document
- Headings must start at line beginning (no indentation)
- Files must end with exactly one newline

### 4. Invisible Characters Matter

**CRITICAL:** Invisible characters can break markdown parsing in ways that are
extremely difficult to debug.

Required character encoding:

- Use ONLY regular spaces (U+0020) for indentation, never tabs or non-breaking
  spaces
- Non-breaking spaces (U+00A0, `&nbsp;`) break markdown parsing
- Ensure proper character encoding (UTF-8)
- Watch for zero-width characters that can break parsing

## Pre-Generation Checklist

Before generating ANY markdown, mentally review:

- [ ] Where will lists appear? (Plan blank lines before/after)
- [ ] Where will headings appear? (Plan blank lines before/after)
- [ ] Where will code blocks appear? (Plan blank lines before/after)
- [ ] What heading levels are needed? (Verify progression)
- [ ] What list style will I use? (Stick to `-` throughout)
- [ ] Does the content need a language identifier for code? (Always specify)
- [ ] Am I showing markdown examples with code blocks? (Use four backticks)
- [ ] Are any lines too long? (Keep under 80 characters when possible)
- [ ] Will I need line breaks within paragraphs? (Use two trailing spaces)
- [ ] Will I use proper spacing? (Regular spaces only, no nbsp or tabs)
- [ ] Will tables be needed? (Use consistent column spacing style)

## Generation Rules

### Rule 1: Lists (MD032, MD004)

**ALWAYS:**

- Add blank line BEFORE list
- Add blank line AFTER list
- Use consistent markers (use `-` exclusively)
- Maintain proper indentation for nested items

**Template:**

```markdown
Preceding paragraph text.

- First item
- Second item
- Third item

Following paragraph text.
```

**NEVER:**

```markdown
Text before
- Item 1
- Item 2
Text after
```

### Rule 2: Headings (MD001, MD003, MD022, MD023)

**ALWAYS:**

- Add blank line BEFORE heading (except at document start)
- Add blank line AFTER heading
- Use ATX style (`#` prefix)
- Include space after `#`
- Start at line beginning (no indentation)
- Increment levels by one only

**Template:**

```markdown
Previous paragraph.

## Section Heading

Content following the heading.

### Subsection

More content here.
```

**NEVER:**

```markdown
Text
## Heading
Text

# Main
### Subsection (skipped level 2!)
```

### Rule 3: Code Blocks (MD031, MD040)

**ALWAYS:**

- Add blank line BEFORE code block
- Add blank line AFTER code block
- Specify language identifier
- Use ``` for fencing (not ~~~)
- Use more backticks for nested examples (see below)

**Template:**

````markdown
Here's an example:

```python
def hello():
    print("Hello, World!")
```

The code demonstrates...
````

**NEVER:**

````markdown
Here's an example:
```
code here
```
More text.
````

#### Nested Code Block Rule (CRITICAL)

When showing markdown examples that contain code blocks, use **one more
backtick** than the deepest nested level:

- **Three backticks (```)**: Regular code blocks
- **Four backticks (````)**: Showing markdown with code blocks
- **Five backticks (`````)**: Showing markdown showing markdown with code
  blocks

**Example - Wrong (causes MD040):**

<!-- markdownlint-disable MD031 MD040 -->

````markdown
```markdown
# Example Document

```bash
command here
```
```
````

<!-- markdownlint-enable MD031 MD040 -->

**Example - Right:**

`````markdown
````markdown
# Example Document

```bash
command here
```

````
`````

The parser needs the extra backtick to distinguish the outer fence from inner
fences.

### Rule 4: Line Length (MD013)

**ALWAYS:**

- Keep lines under 80 characters when possible
- Break long paragraphs at natural points
- Split long sentences across multiple lines
- Watch for long URLs and file paths

**When Line Length Matters:**

- Prose text and paragraphs
- Headings and titles
- List items
- Link text

**When Line Length Is Often Ignored:**

- Code blocks (usually excluded by default)
- URLs that cannot be shortened
- Tables with required width
- HTML tags and comments

**Breaking Long Lines:**

```markdown
Wrong (95 characters):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RedondoK/claude-md-skill](https://github.com/RedondoK/claude-md-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
