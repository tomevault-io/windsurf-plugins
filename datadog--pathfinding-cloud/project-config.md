---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## When to Use Each File

- **CLAUDE.md** (this file): Workflow guidance, commands, development tasks, and quick references
- **SCHEMA.md**: Authoritative field definitions, validation rules, and format specifications
- **.claude/CLAUDE.md**: Anti-patterns, style guide, and contribution-specific guidance

**Rule of thumb**: Check SCHEMA.md for "what does field X mean?", check this file for "how do I do task Y?"

## Project Overview

**pathfinding.cloud** is a comprehensive, community-maintained library documenting AWS IAM privilege escalation paths. The project consists of:

1. **Data Layer**: Structured YAML files documenting each privilege escalation path
2. **Validation Layer**: Python scripts to validate YAML against schema
3. **Website Layer**: Static HTML/CSS/JS site for browsing paths
4. **CI/CD Layer**: GitHub Actions for validation and deployment

## Terminology: Parent/Child vs Primary/Variant

We use **different terminology in different contexts** for clarity and semantic meaning:

### In YAML Files and Code
- **`parent` field**: Points to the parent path (e.g., `parent.id: iam-002`)
- **Why**: Concise, follows common data structure conventions, natural for hierarchical references

### In UI and Documentation
- **"Primary Technique"**: The foundational/original technique (what YAML calls the "parent")
- **"Variant"**: A modification that expands applicability by removing prerequisites (what YAML calls the "child")
- **Why**: Semantic clarity - "variant" explains WHAT it is, not just that there's a hierarchy

### Key Concepts
- **Primary techniques** have no `parent` field - they are the foundational attacks
- **Variant techniques** have a `parent` field with `id` and `modification`
- **Variants add required permissions** that remove prerequisites from the primary technique
- **Example**: IAM-002 (primary) requires < 2 keys. IAM-003 (variant) adds DeleteAccessKey to work even with 2 keys.

### When Contributing
- In YAML: Use `parent` field for variants
- In documentation/comments: Refer to "primary techniques" and "variants"
- In UI text: Display "Primary Technique" and "Variants (N)"

See [SCHEMA.md](SCHEMA.md#parent-object-optional) for detailed parent/child relationship criteria.

## Quick Start Commands

### Essential Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Validate a single file
python scripts/validate-schema.py data/paths/{service}/{service}-###.yaml

# Validate all files
python scripts/validate-schema.py data/paths/

# Generate JSON for website
python scripts/generate-json.py

# Test website locally (with SPA routing support)
cd docs && python3 dev-server.py
# Then visit http://localhost:8888
```

### GitHub Token for Better Contributor Info

```bash
# Set GitHub token (optional but recommended)
export GITHUB_TOKEN=your_github_pat_here
python scripts/generate-json.py

# Token scopes:
# - Private repos: 'repo' scope
# - Public repos: 'public_repo' scope
# Create at: https://github.com/settings/tokens/new
```

## Architecture

### Data Structure

All privilege escalation paths are stored as individual YAML files in `data/paths/{service}/`:
- Files follow naming convention: `{service}-{number}.yaml` (e.g., `iam-001.yaml`)
- Each file adheres to the schema defined in [SCHEMA.md](SCHEMA.md)
- Files are organized by primary service (iam, ec2, lambda, ssm, cloudformation, etc.)

### ID Numbering Convention

- **IAM-focused paths**: `iam-001`, `iam-002`, etc.
- **PassRole combinations**: Use the service of the resource being created/manipulated
  - `iam:PassRole+ec2:RunInstances` → `ec2-001` (not iam-###)
  - `iam:PassRole+lambda:CreateFunction` → `lambda-001`
- **Other services**: `ssm-001`, `ec2-002`, etc.
- **Sequential numbering**: IDs are assigned sequentially within each service

### Website Architecture (SPA with Client-Side Routing)

The website is a Single Page Application (SPA) with client-side routing:
- **List view**: `/` - Shows all paths with search/filter functionality
- **Detail view**: `/paths/{id}` - Shows individual path details (e.g., `/paths/iam-001`)
- **Routing**: Uses History API (`pushState`/`popState`) for proper URLs
- **No page reloads**: Navigation is instant, only content changes
- **SEO ready**: Dynamic meta tags per page, Open Graph support
- **Analytics ready**: Real pageviews on route changes
- **Backward compatible**: Old hash URLs (`#iam-001`) redirect to new format

**Directory Structure:**
- All website files are in the `docs/` directory (GitHub Pages compatible)
- Source data (YAML files) remain at `data/paths/` in repository root
- Generated files (`paths.json`, `metadata.json`) are created in `docs/`

**Development:**
- Use `docs/dev-server.py` for local testing (handles SPA routing)
- Run from project root: `cd docs && python3 dev-server.py`
- Direct file opening won't support routing features

**Production (GitHub Pages):**
- GitHub Pages deploys only the `docs/` directory
- `404.html` implements the SPA routing pattern for GitHub Pages
- When users access direct URLs (e.g., `/paths/iam-001`), GitHub Pages serves `404.html`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DataDog/pathfinding.cloud](https://github.com/DataDog/pathfinding.cloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
