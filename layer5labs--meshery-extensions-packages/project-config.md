---
trigger: always_on
description: This document provides comprehensive instructions for GitHub Copilot coding agents working on the [meshery-extensions-packages](https://github.com/meshery-extensions/meshery-extensions-packages) repository.
---

# GitHub Copilot Coding Agent Instructions for meshery-extensions-packages

This document provides comprehensive instructions for GitHub Copilot coding agents working on the [meshery-extensions-packages](https://github.com/meshery-extensions/meshery-extensions-packages) repository.

## Repository Overview

The `meshery-extensions-packages` repository is a multi-function repository that serves several critical purposes in the Layer5/Meshery ecosystem:

### Primary Functions

1. **Layer5 Badges** - Recognition program assets (SVG/PNG format)
2. **Layer5 Cloud Remote Provider Packages** - Email templates and release artifacts
3. **Kanvas Snapshot Images** - Infrastructure-as-code visual diagrams stored as images
4. **Embedded Meshery Designs** - Exportable design files with HTML/CSS/JS

### Repository Structure

```
/assets
  /badges           - SVG and PNG badge representations
  /meshmap          - Animated GIFs, SVGs, quick tips
  /organizations    - Custom org icons (PNG) namespaced by org UUID
  /kanvas           - Images for Kanvas designs, roadmaps, flowcharts
    /getting-started - WebP format images
/email              - HTML email templates, assets for layer5-cloud
/action-assets      - PR snapshot images (namespaced by year/month)
  /2024@7/<design-id>-light.png
  /2024@7/<design-id>-dark.png
/design-assets      - Catalog item snapshots (permanent links)
  /<design-id>-light.png
  /<design-id>-dark.png
/meshery-design-embed - Exported design files for web embedding
```

## Critical Considerations

### ⚠️ SPARSE CHECKOUT REQUIREMENT

**IMPORTANT**: This repository has a very large file size due to extensive image and asset storage. Always use sparse checkout to avoid overloading GitHub Action runners.

```bash
# Correct sparse checkout approach
git clone --filter=blob:none --sparse https://github.com/meshery-extensions/meshery-extensions-packages
git sparse-checkout add assets
```

**For CI/CD workflows, always configure sparse checkout:**

```yaml
- uses: actions/checkout@v4
  with:
    repository: meshery-extensions/meshery-extensions-packages
    sparse-checkout: |
      assets/badges
      email
    sparse-checkout-cone-mode: false
```

### Cross-Repository Integration

The meshery-extensions-packages repository integrates with multiple related repositories:

1. **[meshery/meshery](https://github.com/meshery/meshery)** - Main Meshery application
   - Cloud native infrastructure management platform
   - Kubernetes multi-cluster deployments
   - Visual and collaborative GitOps

2. **[layer5io/meshery-cloud](https://github.com/layer5io/meshery-cloud)** - Layer5 Cloud backend
   - Remote provider for Meshery
   - Authentication and user management
   - Catalog and design storage

3. **[layer5labs/meshery-extensions](https://github.com/layer5labs/meshery-extensions)** - Meshery UI extensions
   - Extension framework for Meshery
   - UI components and plugins

4. **[layer5labs/kanvas-snapshot](https://github.com/layer5labs/kanvas-snapshot)** - GitHub Action
   - Generates visual snapshots of infrastructure-as-code
   - Supports Kubernetes manifests, Helm charts, Docker Compose
   - Uploads snapshots to meshery-extensions-packages

## Coding Best Practices

### 1. Asset Management

**When adding or modifying assets:**

- Follow established naming conventions:
  - PR snapshots: `/action-assets/YYYY@M/<design-id>-{light|dark}.png`
  - Catalog snapshots: `/design-assets/<design-id>-{light|dark}.png`
  - Organization icons: `/assets/organizations/<org-uuid>/{mobile|desktop}.png`

- Always provide both light and dark mode versions for images
- Use appropriate formats:
  - Badges: SVG (preferred) and PNG
  - Screenshots: PNG
  - Marketing/UI: WebP for better compression
  - Animated content: GIF or animated SVG

- Optimize images before committing:
  ```bash
  # Example using imagemagick
  convert input.png -strip -quality 85 output.png
  ```

### 2. Email Template Development

**When working with email templates in `/email`:**

- Use inline CSS for email compatibility
- Test across multiple email clients (Gmail, Outlook, Apple Mail)
- Ensure images are referenced via permanent URLs (not relative paths)
- Include alt text for all images
- Use table-based layouts for maximum compatibility
- Test responsive behavior for mobile devices

### 3. CI/CD Workflow Development

**When creating or modifying GitHub Actions workflows:**

- Always use sparse checkout (see example above)
- Set appropriate timeouts to prevent runaway jobs
- Use caching for dependencies when possible
- Handle large file uploads in chunks if needed
- Implement proper error handling and retry logic

```yaml
# Example workflow with sparse checkout
name: Update Badges
on:
  workflow_dispatch:
  push:
    branches: [master, main]
    paths:
      - 'assets/badges/**'

jobs:
  process-badges:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
        with:
          sparse-checkout: |
            assets/badges
          sparse-checkout-cone-mode: false
          
      - name: Process badges
        run: |
          # Your badge processing logic here

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [layer5labs/meshery-extensions-packages](https://github.com/layer5labs/meshery-extensions-packages) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
