---
trigger: always_on
description: This skill should:
---

# CLAUDE.md - Contributor Guide

> **For End Users:** See [README.md](README.md) for installation and usage.
>
> **This file** is for contributors, maintainers, and skill developers.

## What This Is

This repository contains a **Claude Code skill** - executable documentation that Claude loads to provide Terraform/OpenTofu expertise. Think of it as:

- **Prompt engineering as infrastructure**: Version-controlled AI instructions
- **Domain knowledge artifact**: Encoding terraform-best-practices.com into Claude's context
- **Meta-project**: Building instructions for an AI assistant

## Repository Structure

```
terraform-skill/
├── .claude-plugin/
│   └── marketplace.json                  # Marketplace and plugin metadata
├── SKILL.md                              # Core skill file (~524 lines)
├── references/                           # Reference files (progressive disclosure)
│   ├── ci-cd-workflows.md                # CI/CD templates (~473 lines)
│   ├── code-patterns.md                  # Code patterns & modern features (~859 lines)
│   ├── module-patterns.md                # Module best practices (~1,126 lines)
│   ├── quick-reference.md                # Command cheat sheets (~600 lines)
│   ├── security-compliance.md            # Security guidance (~470 lines)
│   └── testing-frameworks.md             # Testing guides (~563 lines)
├── README.md                             # For GitHub/marketplace users
├── CLAUDE.md                             # For contributors (YOU ARE HERE)
└── LICENSE                               # Apache 2.0
```

### File Roles

| File | Audience | Purpose |
|------|----------|---------|
| `.claude-plugin/marketplace.json` | Claude Code | Marketplace and plugin metadata |
| `SKILL.md` | Claude Code | Core skill (~524 lines, ~4.4K tokens) |
| `references/*.md` | Claude Code | Reference files loaded on demand (6 files, ~26K tokens) |
| `README.md` | End users | Installation, usage examples, what it covers |
| `CLAUDE.md` | Contributors | Development guidelines, architecture decisions |
| `LICENSE` | Everyone | Apache 2.0 legal terms |

## How Claude Skills Work

### Progressive Disclosure

```
User: "Create a Terraform module with tests"
       ↓
Claude: Scans skill metadata (~100 tokens)
       ↓
Claude: "This matches terraform-skill activation triggers"
       ↓
Claude: Loads full SKILL.md (~4,400 tokens)
       ↓
Claude: Applies testing framework decision matrix
       ↓
Response: Code following best practices
```

**Key Insight:** Skills only load when relevant, minimizing token usage.

### Token Budget

- **Metadata (YAML frontmatter):** ~100 tokens - always loaded
- **Core SKILL.md:** ~4,400 tokens - loaded on activation
- **Reference files:** Individual estimates (loaded on demand only):
  - ci-cd-workflows.md: ~2,300 tokens
  - code-patterns.md: ~5,100 tokens
  - module-patterns.md: ~7,000 tokens
  - quick-reference.md: ~3,800 tokens
  - security-compliance.md: ~2,500 tokens
  - testing-frameworks.md: ~3,400 tokens
- **Target:** Aim for under 500 lines for main SKILL.md (current: 524 lines - comprehensive core guidance)

**Our Architecture:**
- SKILL.md: 524 lines, ~4.4K tokens (comprehensive core guidance)
- Reference files: 6 files totaling 4,091 lines, ~26K tokens
- Progressive disclosure: ~56-70% token reduction for typical queries (vs loading all content)

## Content Philosophy

### What Belongs in SKILL.md

✅ **Include:**
- Terraform-specific patterns and idioms
- Decision frameworks (when to use X vs Y)
- Version-specific features (Terraform 1.6+, 1.9+, etc.)
- Testing strategy workflows
- ✅ DO vs ❌ DON'T examples
- Quick reference tables and decision matrices

✅ **Keep:**
- Scannable format (tables, headers, visual hierarchy)
- Imperative voice ("Use X", not "You should consider X")
- Concrete examples with inline comments
- Version requirements clearly marked

### What Doesn't Belong

❌ **Exclude:**
- Generic programming advice
- Terraform syntax basics (covered in official docs)
- Provider-specific resource details (use MCP tools)
- Obvious practices ("use version control")
- Long prose explanations (use tables instead)

## Content Structure

SKILL.md is organized by workflow phase:

1. **When to Use This Skill** - Activation triggers for Claude
2. **Core Principles** - Naming, structure, philosophy
3. **Testing Strategy Framework** - Decision matrices
4. **Module Development** - Best practices and patterns
5. **Common Patterns** - ✅/❌ side-by-side examples
6. **CI/CD Integration** - Workflow automation
7. **Quick Reference** - Rapid consultation tables
8. **License & Attribution** - Legal and source credits

Each section is self-contained for selective reading.

## Writing Style Guide

### Imperative Voice

✅ **Good:**
```markdown
Use underscores in variable names, not hyphens:

✅ DO: `variable "vpc_id" {}`
❌ DON'T: `variable "vpc-id" {}`
```

❌ **Bad:**
```markdown
You should consider using underscores instead of hyphens
in your variable names, as this is generally preferred.
```

### Scannable Format

Use:
- **Tables** for comparisons and decision matrices
- **Code blocks** with inline comments
- **Headers** for clear section breaks
- **Bullets** for lists, not paragraphs
- **✅/❌** for visual clarity

### Version Requirements

Always mark version-specific features:

```markdown

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antonbabenko/terraform-skill](https://github.com/antonbabenko/terraform-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
