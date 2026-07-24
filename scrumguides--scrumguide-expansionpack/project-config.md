---
trigger: always_on
description: This guide provides instructions for AI assistants (like GitHub Copilot) working with the Scrum Guide Expansion Pack codebase. It outlines the project structure, deployment model, and best practices for assisting developers.
---

# 🤖 AI Agent Guide

This guide provides instructions for AI assistants (like GitHub Copilot) working with the Scrum Guide Expansion Pack codebase. It outlines the project structure, deployment model, and best practices for assisting developers.

## Project Overview

The **Scrum Guide Expansion Pack** is a Hugo-based static website hosted on Azure Static Web Apps. It provides modern guidance for applying Scrum to complex work, AI, and adaptive strategy.

**Live Sites:**

- **Production**: [scrumexpansion.org](https://scrumexpansion.org)
- **Preview**: [agreeable-island-0c966e810-preview.centralus.6.azurestaticapps.net](https://agreeable-island-0c966e810-preview.centralus.6.azurestaticapps.net/)

## 📚 Essential Documentation

Before assisting with any task, familiarize yourself with these key documents:

| Document | Purpose | Reference When |
|----------|---------|----------------|
| [Getting Started](./docs/getting-started.md) | Initial setup and installation | Setting up development environment |
| [Development Guide](./docs/development.md) | Development workflows and standards | Writing code, creating content, or templates |
| [Deployment Guide](./docs/deployment.md) | Deployment process and environments | Deployment questions or CI/CD issues |
| [Content Management](./docs/content-management.md) | Content creation and organization | Creating or editing content |
| [Contributing Guide](./docs/contributing.md) | Contribution guidelines | Pull request process or contribution questions |
| [Configuration](./docs/configuration.md) | Hugo and Azure configuration | Configuration changes needed |
| [Translations](./docs/translations.md) | Internationalization process | Adding or updating translations |
| [Troubleshooting](./docs/troubleshooting.md) | Common issues and solutions | Debugging problems |

## 🚀 Deployment Model

Understanding the deployment model is **critical** for proper development workflow:

### 1. Development Phase

**Workflow:**

```
Fork/Branch → Make Changes → Local Testing → Create Pull Request
```

**Key Points:**

- Developers work in **feature branches** or **forks**
- All changes must be **tested locally** before creating a PR
- Follow the branching strategy in [Contributing Guide](./docs/contributing.md)

### 2. Pull Request Testing

**What Happens:**

```
Pull Request Created → Automatic deployment to PR-specific test site
```

**PR Environment Details:**

- **URL Pattern**: `https://agreeable-island-0c966e810-{PullRequestId}.centralus.6.azurestaticapps.net`
- **Example**: PR #42 → `https://agreeable-island-0c966e810-42.centralus.6.azurestaticapps.net`
- **Purpose**: Test changes in isolation before merging
- **Lifecycle**: Automatically created when PR opens, removed when PR closes
- **Important**: Only PRs to the main repository get deployed (not from forks)

### 3. Preview Deployment

**What Happens:**

```
PR Merged to Main → Automatic deployment to Preview environment
```

**Preview Environment Details:**

- **URL**: [agreeable-island-0c966e810-preview.centralus.6.azurestaticapps.net](https://agreeable-island-0c966e810-preview.centralus.6.azurestaticapps.net/)
- **Branch**: `main`
- **Purpose**: Pre-production testing and validation
- **Audience**: Internal team and stakeholders
- **Configuration**: `staticwebapp.config.preview.json` + `hugo.preview.yaml`

### 4. Production Deployment

**What Happens:**

```
Create GitHub Release → Tag with version → Automatic deployment to Production
```

**Production Deployment Details:**

- **URL**: [scrumexpansion.org](https://scrumexpansion.org)
- **Trigger**: Creating a **GitHub Release** with a **version tag**
- **Version Strategy**: Semantic versioning
- **Configuration**: `staticwebapp.config.production.json` + `hugo.yaml`

## 📋 Version Numbering Strategy

When deploying to production via GitHub releases, follow **semantic versioning** principles:

### Version Format: `vMAJOR.MINOR.PATCH`

| Type | When to Use | Example |
|------|-------------|---------|
| **Patch** (`v1.0.1`) | Tiny changes: typo fixes, small corrections, minor tweaks | `v1.0.1` → Fix typo in guide |
| **Minor** (`v1.1.0`) | Minor changes: new section, content additions, feature additions | `v1.1.0` → Add new guide section |
| **Major** (`v2.0.0`) | Major changes: complete document revamp, breaking changes, restructure | `v2.0.0` → Complete guide overhaul |

### Creating a Release

**Steps:**

1. Navigate to **GitHub → Releases → Draft a new release**
2. Choose or create a **tag** (e.g., `v1.2.0`)
3. Select **target branch** (usually `main`)
4. Add **release title** and **description**
5. Click **Publish release**
6. Automatic deployment to production begins

**Example Release Notes:**

```markdown
## v1.2.0 - New Psychological Safety Guide

### Added
- New guide: Psychological Safety in Scrum Teams
- Translation support for German (DE)

### Changed
- Updated homepage layout
- Improved mobile navigation

### Fixed
- Corrected references in Complexity guide
```

## 🛠️ Technology Stack

### Core Technologies

- **Hugo Extended** (v0.146.0+) - Static site generator

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ScrumGuides/ScrumGuide-ExpansionPack](https://github.com/ScrumGuides/ScrumGuide-ExpansionPack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
