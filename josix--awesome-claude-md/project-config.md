---
trigger: always_on
description: This file provides guidance to GitHub Copilot when working with code in this repository.
---

# GitHub Copilot Instructions

This file provides guidance to GitHub Copilot when working with code in this repository.

**Important**: This file should stay synchronized with `CLAUDE.md` to ensure consistent AI assistant behavior across different tools. Any changes to project structure, guidelines, or standards should be reflected in both files.

## Project Overview

**awesome-claude-md** is a curated collection of high-quality `CLAUDE.md` files from public GitHub repositories, plus a curated list of tools that generate, sync, or manage `CLAUDE.md` files. The goal is to showcase best practices for using `CLAUDE.md` files to onboard AI assistants to codebases.

## Repository Structure

When suggesting file paths or navigation, follow this structure:
```
awesome-claude-md/
├── README.md                    # Main landing page with table of contents
├── CLAUDE.md                    # Project guidance for Claude Code
├── .github/
│   └── copilot-instructions.md  # This file
├── docs/                        # GitHub Pages static site
│   ├── _config.yml              # Jekyll configuration
│   ├── _layouts/                # HTML layouts
│   ├── assets/                  # CSS and JavaScript
│   │   ├── css/style.css        # Dark-themed responsive styles
│   │   └── js/main.js           # Search and filter functionality
│   └── index.html               # Main browsable page
└── scenarios/                   # Categorized examples
    ├── [category]/
    │   └── [owner]_[repo]/
    │       └── README.md        # Analysis with links to original files
```

## Core Categories

When adding new examples, use these primary categories:
- `complex-projects/` - Multi-service projects with detailed architecture
- `libraries-frameworks/` - Core concepts, APIs, and usage patterns
- `developer-tooling/` - CLI tools with commands and configuration
- `project-handoffs/` - Current state with blocking issues and next steps
- `getting-started/` - Development environment setup focused
- `infrastructure-projects/` - Large-scale systems and runtime environments

## Tools & Ecosystem

The repository also maintains a curated table of tools directly related to `CLAUDE.md` workflows (e.g., tools that generate, sync, or manage `CLAUDE.md` files). These are listed in the `README.md` under the "Tools & Ecosystem" section and are NOT counted as examples. Inclusion criteria: must be directly related to `CLAUDE.md` workflows. License type is shown for reference.

## File Naming Conventions

### Directory Names
- Use lowercase with hyphens: `complex-projects`, `developer-tooling`
- Repository directories: `[owner]_[repo]` format (e.g., `microsoft_semanticworkbench`)

### File Names
- Original files: `CLAUDE.md` (preserve exactly as found)
- Analysis files: `README.md` (our evaluation)
- This displays automatically when browsing GitHub directories

## Common Development Patterns

### Documentation Structure
When creating README.md files, follow this pattern:
```markdown
# Analysis: [Project Name]

**Category**: [Category Name]
**Repository**: [GitHub URL]
**Why it's exemplary**: [Brief explanation]

## Key Features That Make This Exemplary

### 1. **[Feature Name]**
- **[Aspect]**: [Description]
- **[Commands/Examples]**: [Specific examples]

## Specific Techniques to Learn

### [Technique Name]
```[language]
[Code example]
```
[Explanation]

## Key Takeaways

1. **[Takeaway 1]**: [Description]
2. **[Takeaway 2]**: [Description]
```

### README.md Updates
When adding new examples, update the main README.md:
- Add entry in appropriate category section
- Include project name, repository link, and star count
- List 2-3 key features in bullet points
- Link to the README.md file

### Search Commands
For finding new examples, suggest these GitHub search patterns:

#### Manual Search Queries
```
filename:CLAUDE.md "## Architecture"
filename:CLAUDE.md "## Development Commands"
"## Testing" filename:CLAUDE.md
"## Deployment" filename:CLAUDE.md
filename:CLAUDE.md language:TypeScript
```

#### KOL and Expert Organization Search
Target repositories from key opinion leaders and expert organizations:
```
filename:CLAUDE.md user:anthropics       # AI experts and Claude creators
filename:CLAUDE.md user:pydantic         # Python validation library experts
filename:CLAUDE.md user:microsoft        # Enterprise AI and infrastructure
filename:CLAUDE.md user:gaearon          # React co-creator Dan Abramov
filename:CLAUDE.md user:openai           # AI research and development
filename:CLAUDE.md user:cloudflare       # Infrastructure and runtime systems
filename:CLAUDE.md user:pytorch          # Machine learning frameworks
```

#### Domain-Specific Searches
```
# Python Ecosystem
filename:CLAUDE.md user:fastapi OR user:tiangolo OR user:pydantic

# JavaScript/React
filename:CLAUDE.md user:vercel OR user:facebook OR user:nextjs

# AI/ML
filename:CLAUDE.md user:huggingface OR user:langchain-ai

# Infrastructure
filename:CLAUDE.md user:docker OR user:kubernetes
```

## Quality Standards

### Ethical Guidelines
- **Never copy** `CLAUDE.md` files directly into this repository
- **Always link** to the original source repository
- **Include attribution** with source links, licensing information, and proper credit

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [josix/awesome-claude-md](https://github.com/josix/awesome-claude-md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
