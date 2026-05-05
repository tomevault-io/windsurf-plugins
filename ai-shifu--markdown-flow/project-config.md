---
trigger: always_on
description: _This document is shared by `CLAUDE.md`, `AGENTS.md`, and `GEMINI.md`._
---

# MarkdownFlow Agent Guide

_This document is shared by `CLAUDE.md`, `AGENTS.md`, and `GEMINI.md`._

Follow these notes alongside any agent-specific instructions provided by the hosting platform.

## Project Overview

MarkdownFlow is a Markdown format extension that enables AI-powered, personalized, and interactive documents. The tagline is "Write Once, Deliver Personally" (一次创作，千人千面). Documents written in MarkdownFlow can be processed by MarkdownFlow Agents to create well-formatted, personalized pages tailored to each reader.

## Repository Structure

This is the main documentation and landing page repository for the MarkdownFlow ecosystem. It contains:

- `home/index.html` - Homepage with project overview and examples
- `docs/` - MkDocs-based documentation site
- `.github/workflows/` - GitHub Actions for automated deployment

The actual MarkdownFlow implementations are in separate repositories under the ai-shifu organization:

- Frontend: markdown-flow-ui, remark-flow, markdown-it-flow
- Backend: markdown-flow-agent-py, markdown-flow-agent-go

## Common Commands

### Documentation Development

```bash
# Install MkDocs dependencies
pip install mkdocs-material pymdown-extensions

# Run MkDocs locally
cd docs
mkdocs serve

# Build documentation
mkdocs build --site-dir site --clean
```

### Pre-commit Hooks

```bash
# Install pre-commit
pip install pre-commit

# Install hooks
pre-commit install

# Run manually on all files
pre-commit run --all-files
```

### Deployment

The site automatically deploys to GitHub Pages when pushing to main branch via `.github/workflows/deploy-gh-pages.yml`. The workflow:

1. Copies the static homepage from `home/` and shared assets into `_site/`
2. Builds MkDocs documentation to `_site/docs/`
3. Deploys to GitHub Pages

## MarkdownFlow Core Principle

**IMPORTANT**: MarkdownFlow documents are written **for AI agents, not for human readers**. This is the fundamental paradigm that must be understood and applied throughout all documentation and examples.

- Content in MarkdownFlow documents serves as **instructions/prompts for AI** to generate personalized content
- Examples should demonstrate **AI instructions**, not direct user-facing content
- Traditional approach: "Welcome, John!" (content for humans)
- MarkdownFlow approach: "Generate a welcoming message for {{user_name}}" (instruction for AI)

## MarkdownFlow Syntax

The core MarkdownFlow syntax extensions:

1. **Variables**: `{{variable_name}}` - Dynamic content placeholders
2. **User Input**: `?[%{{variable}}Option1|Option2]` - Interactive buttons that store user choices
3. **Preserved Content**: `===content===` - Content that should not be interpreted by AI

## Documentation Guidelines

When updating documentation:

- Main documentation entry point is `docs/docs/en/index.md`
- Use formal, official tone for documentation
- **ALL examples must follow MarkdownFlow core principle**: Show AI instructions, not user content
- Examples should demonstrate practical use cases with proper "writing for AI" approach
- Navigation structure is defined in `docs/mkdocs.yml`
- Ensure consistency with the paradigm: "You write prompts for AI, not content for humans"

## Linting and Formatting

The repository uses:

- **markdownlint** for Markdown linting (config: `.markdownlint.yaml`)
- **prettier** for Markdown formatting
- **pre-commit** hooks that run automatically on commit

Key linting rules:

- Use ATX-style headings (`#`)
- Use dashes for unordered lists (`-`)
- Use fenced code blocks
- Line length limit is disabled (MD013: false)

## Important Files

- `.github/workflows/deploy-gh-pages.yml` - GitHub Pages deployment configuration
- `docs/mkdocs.yml` - MkDocs configuration and navigation structure
- `.pre-commit-config.yaml` - Pre-commit hooks configuration
- `.markdownlint.yaml` - Markdown linting rules

## Translation Glossary (Chinese)

This glossary provides standardized Chinese translations for MarkdownFlow terminology. It should be used as reference for all Chinese localization work to ensure consistency.

### Core Brand Terms

| English | Chinese | Notes |
|---------|---------|-------|
| MarkdownFlow | MarkdownFlow | Keep English brand name |
| MDFlow | MDFlow | Keep English abbreviation |
| MDF | MDF | Keep English abbreviation |

### Core Concepts

| English | Chinese | Notes |
|---------|---------|-------|
| Write Once, Deliver Personally | 一次创作，千人千面 | Project tagline |
| Write prompts for AI, not content for humans | 为 AI 写提示词，而非为人类写内容 | Core principle |
| AI-powered personalized documents | AI 驱动的个性化文档 | |
| Instructions for AI | AI 指令 | |

### Syntax Elements

| English | Chinese | Notes |
|---------|---------|-------|
| Variables | 变量 | `{{variable}}` |
| Interactive Elements | 交互元素 | |
| Button| 按钮| `?[%{{var}}Option]` |
| Input | 输入框| `?[%{{var}}...input hint]` |
| Preserved Content | 确定内容 | `===content===` |
| Preserved Output | 确定输出| Same as above |

### Document Structure

| English | Chinese | Notes |
|---------|---------|-------|
| Content Prompt | 内容提示词 | Document body |
| Document Prompt | 文档提示词 | Global settings |
| Template | 模板 | |
| Block | 块 | |
| Content Block | 内容块 | |
| Interaction Block | 交互块 | |
| Interaction | 交互 | |
| Preserved Content |  确定内容 | |

### Technical Components


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ai-shifu/markdown-flow](https://github.com/ai-shifu/markdown-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
