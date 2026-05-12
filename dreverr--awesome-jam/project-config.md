---
trigger: always_on
description: This document provides guidelines for AI coding agents working on the Awesome JAM repository. This is a curated list repository containing Markdown documentation, not a traditional code project.
---

# AGENTS.md - Guidelines for AI Coding Agents

This document provides guidelines for AI coding agents working on the Awesome JAM repository. This is a curated list repository containing Markdown documentation, not a traditional code project.

## Repository Overview

**Type**: Curated list / Documentation repository
**Primary Language**: Markdown
**Purpose**: A curated collection of JAM (Join-Accumulate Machine) resources, tools, examples, tutorials, and more
**Main Files**:
- `README.md` - Main curated list (primary content)
- `CONTRIBUTING.md` - Contribution guidelines
- `LICENSE` - MIT License
- `AGENTS.md` - This file (AI agent guidelines)

## Repository Structure

```
Awesome-JAM/
├── README.md          # Main curated list (primary content)
├── CONTRIBUTING.md    # Contribution guidelines
├── LICENSE            # MIT License
└── AGENTS.md          # This file
```

## README Sections

The main `README.md` is organized into these sections (in order):

1. **About JAM** - Overview of JAM technology
2. **SDKs** - Development kits for building JAM services
3. **Tools** - Development tools, debuggers, playgrounds, utilities
4. **Examples & Demos** - Real-world examples and demo projects
5. **Documentation** - Official docs, specs, technical references
6. **Tutorials** - Step-by-step guides and learning resources
7. **Videos** - Conference talks, tutorials, educational content
8. **Articles** - Blog posts and write-ups, with subsections:
   - Technical Deep Dives
   - Educational & Explainers
   - Vision & Analysis
   - Use Cases & Applications
   - Events & News
   - Developer Resources
9. **Community & Resources** - Community links and additional resources
10. **Contributing** - How to contribute (inline summary)

## Working with This Repository

### Validation Steps

When making changes, validate manually:

1. **Markdown Formatting**:
   - Check that all Markdown syntax is correct
   - Ensure proper heading hierarchy (no skipped levels)
   - Verify lists are properly formatted

2. **Link Validation**:
   - All URLs should be accessible (no 404s)
   - GitHub links should point to valid repositories
   - Use HTTPS for all external links

3. **Awesome List Compliance**:
   - Follow [awesome-manifesto](https://github.com/sindresorhus/awesome/blob/main/awesome.md)
   - Maintain quality over quantity
   - Include the awesome badge in README

4. **Table of Contents**:
   - Ensure the Contents section in README matches the actual sections
   - Anchor links must match heading text exactly

## Content Guidelines

### Entry Format

The standard entry format used throughout the README is:

```markdown
- [Resource Name](url) by [@username](github-profile) - Brief description
```

**Variations by section:**

- **SDKs / Tools / Examples**: Standard format with GitHub profile attribution
- **Documentation**: Standard format; some entries may omit attribution for official specs
- **Tutorials**: Standard format; some entries may omit `by` attribution
- **Videos**: Title links to YouTube/video URL, no `by` attribution (author in description if needed)
  ```markdown
  - [Video Title](youtube-url) - Brief description
  ```
- **Articles**: Include author name (not necessarily GitHub handle) and optional date
  ```markdown
  - [Article Title](url) by Author Name (Month Year) - Brief description
  ```
- **Community & Resources**: Standard format; some entries may omit attribution
  ```markdown
  - [Resource Name](url) - Brief description
  ```

### Structure and Organization

1. **Table of Contents**:
   - Keep in sync with actual sections
   - Update TOC when adding new sections
   - Use consistent anchor link formatting

2. **Sections**:
   - Group related resources together
   - Maintain consistent section naming
   - Use descriptive section headers

3. **Resource Ordering**:
   - Within sections, order entries **alphabetically by resource name**
   - For articles, order alphabetically within each subsection
   - Group article subsections logically (Technical Deep Dives, Educational, etc.)

### Markdown Style

1. **Headings**:
   - Use ATX-style headings (`#`, `##`, etc.) not Setext-style
   - One H1 (`#`) per file (the title)
   - Maintain proper hierarchy (don't skip levels)
   - Add blank line before and after headings

2. **Links**:
   - Use inline links: `[text](url)`
   - Include link title when helpful: `[text](url "title")`
   - Prefer absolute URLs over relative URLs
   - Use HTTPS protocol for all external links

3. **Lists**:
   - Use `-` for unordered lists (not `*` or `+`)
   - Use consistent indentation (2 spaces for nested lists)
   - Add blank line before and after lists
   - For numbered lists, use `1.`, `2.`, etc.

4. **Emphasis**:
   - Use `**bold**` for emphasis (not `__bold__`)
   - Use `*italic*` for lighter emphasis (not `_italic_`)
   - Use `` `code` `` for inline code/technical terms

5. **Line Length**:
   - No strict line length limit
   - Break lines at natural boundaries (sentences, list items)
   - Keep URLs on same line as their link text

### Content Standards

1. **Descriptions**:
   - Keep brief (1-2 sentences)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DrEverr/Awesome-JAM](https://github.com/DrEverr/Awesome-JAM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
