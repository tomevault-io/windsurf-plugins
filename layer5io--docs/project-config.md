---
trigger: always_on
description: **IMPORTANT**: When using GitHub Copilot, always select the most powerful AI model available (e.g., GPT-5.1 Codex, Gemini 3 Pro Claude 4.5, or the latest advanced model) to ensure the most comprehensive, most human-understandable documentation with examples, tips, notes, analogies and so on.
---

# Coding Agent Instructions for Layer5 Docs

## AI Model Selection

**IMPORTANT**: When using GitHub Copilot, always select the most powerful AI model available (e.g., GPT-5.1 Codex, Gemini 3 Pro Claude 4.5, or the latest advanced model) to ensure the most comprehensive, most human-understandable documentation with examples, tips, notes, analogies and so on.

## Project Overview

The Layer5 Docs website is a Hugo-based documentation site that serves as the primary documentation resource for Layer5 products, hosted at https://github.com/layer5io/docs and live at https://docs.layer5.io. It provides comprehensive documentation for Layer5 Cloud, Kanvas, and related products, offering tutorials, guides, and reference materials for users and contributors in the cloud native ecosystem.

## Technology Stack

- **Framework**: Hugo (extended version with SCSS support)
- **Theme**: Docsy (Google's documentation theme)
- **Language**: Go templates, HTML, CSS/SCSS, JavaScript
- **Content**: Markdown with Hugo shortcodes
- **Package Manager**: npm
- **Node Version**: See `.nvmrc`
- **Build System**: Hugo CLI, Make

## Core Principles

### 1. Minimal, Surgical Changes
- Make the **smallest possible changes** to accomplish the goal
- Never delete or modify working code unless absolutely necessary
- Focus on precise, targeted modifications rather than wholesale rewrites
- Preserve existing patterns and conventions unless explicitly changing them

### 2. Code Quality Standards
- Follow the existing code style and patterns in the repository
- Ensure proper indentation and formatting in templates and content
- Write clean, readable, self-documenting code with minimal comments unless necessary for complex logic
- Maintain accessibility standards (WCAG 2.1)

### 3. Testing and Validation
- Always validate changes work before considering them complete
- Build the site and verify rendered content: `make build` or `hugo`
- Run the site locally: `make site` or `hugo server -D -F`
- Test changes incrementally and iteratively

## Project Structure

```
docs/
├── .github/                 # GitHub configuration and workflows
├── assets/                  # Site assets (CSS, JS, images)
├── charts/                  # Chart files
├── content/
│   └── en/                 # English content
│       ├── cloud/          # Layer5 Cloud documentation
│       ├── kanvas/         # Kanvas documentation
│       ├── contributing/   # Contribution guidelines
│       └── videos/         # Video content
├── data/                   # Hugo data files
├── layouts/                # Hugo templates and layouts
│   ├── _default/           # Default layouts
│   ├── partials/           # Partial templates
│   └── shortcodes/         # Custom shortcodes
├── static/                 # Static assets
├── hugo.toml               # Hugo configuration
├── package.json            # npm dependencies
├── Makefile                # Build automation
└── CONTRIBUTING.md         # Contribution guidelines
```

## Development Workflow

1. Don't fork this repo.
1. Create a branch and pull request in this repo.
1. Don't mark your pull request as draft.

### Setup
```bash
# Install dependencies (required for fresh clone)
make setup
# or
npm install
```

### Development
```bash
# Start development server with drafts and future content
make site
# or
hugo server -D -F
```

### Building
```bash
# Build for production
make build
# or
hugo

# Clean and rebuild
make clean
```

### Docker Development
```bash
# Run with Docker (requires Docker Desktop 4.24+ or Docker Compose 2.22+)
make docker
```

## Content Guidelines

### Tone and Style
- Use a **professional yet approachable** tone
- Content should be clear, concise, and welcoming to both technical and non-technical audiences
- Align with Layer5's mission of empowering engineers to "expect more from their infrastructure"
- Use American English spelling and grammar

### Markdown Content
- All documentation content is written in Markdown
- Place content files in appropriate directories under `content/en/`
- Include proper frontmatter with metadata:

```yaml
---
title: "Page Title"
description: "Short description for SEO (150-160 chars)"
weight: 10  # Optional: controls ordering in navigation
---
```

### Hugo Shortcodes
Use the project's custom shortcodes for enhanced content:

```markdown
{{< alert type="success" title="Note" >}} Your Note {{< /alert >}}
```

Alert types:
- `type="danger"`: Critical alerts (security-related or breaking changes)
- `type="info"`: General informational content
- `type="warning"`: Important warnings that need attention
- `type="note"`: Neutral notes and tips
- `type="success"`: Positive outcomes or confirmations

### Image Guidelines
- Use the following syntax: `![alt text](/path/to/image.svg)` or `<img src="" alt="" />`
- Always provide complete image paths for subpages
- Add `data-modal="false"` to prevent images from opening in a modal
- Always include descriptive alt text for accessibility and SEO

### Content Restrictions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [layer5io/docs](https://github.com/layer5io/docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
