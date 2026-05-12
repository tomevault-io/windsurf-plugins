---
trigger: always_on
description: This document provides guidance for GitHub Copilot agents and Large Language
---

# GitHub Copilot Agents for Layer5 Academy

This document provides guidance for GitHub Copilot agents and Large Language
Models (LLMs) contributing to the Layer5 Academy content repository at
<https://github.com/layer5io/layer5-academy>. These guidelines ensure
consistency, quality, and alignment with the project's educational mission.

## Repository Overview

**Layer5 Academy** is the **official content repository** for Layer5's learning
platform, hosting all official learning paths, challenges, and certifications.
It serves as the primary source of educational content for engineers learning
about cloud native infrastructure, service meshes, Kubernetes, and related
technologies.

**Key Characteristics:**

- **Role**: Content repository for Layer5 Academy learning platform
- **Primary Technology**: Hugo static site generator (extended version 0.146.0+)
- **Content Focus**: Educational materials including learning paths,
  challenges, certifications, and hands-on labs
- **Integration**: Works in combination with:
  - [academy-theme](https://github.com/layer5io/academy-theme) - provides
    layouts, styles, and components
  - [academy-build](https://github.com/layer5io/academy-build) - central build
    and deployment pipeline
  - [academy-example](https://github.com/layer5io/academy-example) - starter
    template for organizations

**Technology Stack:**

- Hugo (extended version 0.146.0 or later)
- Go modules (for theme management)
- Node.js & npm (for build tools)
- Markdown/MDX for content
- PostCSS for styling

## Development Environment

### Prerequisites

- **Go** (version 1.24.5 or later) - Required for Hugo modules
- **Hugo Extended** (version 0.146.0 or later) - Core static site generator
- **Node.js and npm** - For build tools and dependencies
- **Git** - For version control

### Setup Commands

```bash
# Clean up and verify Go module dependencies
go mod tidy

# Install necessary tools and npm packages
make setup

# Start the local Hugo development server
make site
```

The local development server will be available at
`http://localhost:1313/academy`. Note that the local preview uses only the
academy-theme. In production, content is wrapped by the Layer5 Cloud UI, so
minor visual differences may occur.

### Build Commands

```bash
# Build site for production
make build

# Build site for local consumption with custom base URL
make build-preview

# Clean cache and restart development server
make clean

# Fix Markdown linting issues
make lint-fix

# Update academy-theme to latest version
make theme-update
```

## Repository Structure

```bash
layer5-academy/
├── content/              # All learning content (Markdown/MDX)
│   ├── _index.md        # Homepage content
│   ├── certifications/  # Certification tracks
│   ├── challenges/      # Hands-on challenges
│   └── learning-paths/  # Structured learning paths
├── layouts/             # Hugo layout overrides (if any)
├── static/              # Static assets (images, videos, documents)
├── .github/             # GitHub workflows and configuration
│   ├── workflows/       # CI/CD pipelines
│   └── build/           # Build scripts
├── hugo.yaml            # Hugo site configuration
├── go.mod              # Go module dependencies (includes academy-theme)
├── package.json        # npm dependencies
├── Makefile            # Commands for local development & build
└── CONTRIBUTING.md     # Contribution guidelines
```

## Content Creation Guidelines

### Target Audience

Content is specifically tailored for:

- Platform engineers
- DevOps engineers
- Site Reliability Engineers (SREs)
- IT administrators
- Kubernetes operators
- Cloud native developers
- Open source contributors
- Solution architects
- Enterprise architects

### Content Structure

Layer5 Academy supports several educational content types:

1. **Learning Paths** (`/content/learning-paths/`) - Structured sequences of
   educational modules covering comprehensive topics
2. **Challenges** (`/content/challenges/`) - Hands-on practice exercises and
   real-world scenarios
3. **Certifications** (`/content/certifications/`) - Formal credential tracks
   with assessments
4. **Labs** - Interactive, hands-on exercises embedded within learning paths
5. **Modules** - Individual units within learning paths

### Markdown Guidelines

- **Format**: All content must be written in Markdown following Hugo conventions
- **Frontmatter**: Every content file requires proper YAML frontmatter
- **Tone**: Professional yet approachable, clear and concise
- **Style**: Follow technical writing best practices
- **Audience Level**: Adapt complexity to target audience (beginners,
  intermediate, advanced)

### Example Frontmatter

```yaml
---
title: "Getting Started with Service Meshes"
description: "Learn the fundamentals of service mesh architecture and implementation"
date: 2025-01-15
author: "Layer5 Community"
draft: false
weight: 1
tags:
  - service-mesh
  - kubernetes
  - microservices
categories:
  - fundamentals
---
```

### Content Best Practices

1. **Accuracy**: Ensure all technical information is current and correct
2. **Clarity**: Use clear explanations with examples
3. **Practical Focus**: Include hands-on exercises and real-world applications

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [layer5io/layer5-academy](https://github.com/layer5io/layer5-academy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
