---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the Cloudflare Workers Templates repository containing a collection of starter templates for building full-stack applications on the Cloudflare Workers platform. The repository is structured as a monorepo with:

- **Individual template directories** (each ending in `-template`)
- **Custom CLI tool** (`cli/`) for template management and validation
- **Turbo build system** for managing multiple projects
- **Playwright E2E tests** for template validation

## Essential Commands

### Development Commands

```bash
# Check all templates and code quality
pnpm run check

# Fix formatting and linting issues automatically
pnpm run fix

# Run all tests (includes template validation)
pnpm run test

# Run Playwright E2E tests
pnpm run test:e2e

# Run CLI tests specifically
pnpm run test:cli
```

### Template Management Commands

```bash
# Lint all templates for compliance
pnpm run check:templates

# Fix template issues automatically
pnpm run fix:templates

# Generate npm lockfiles for all templates
pnpm run fix:lockfiles

# Validate live demo links
pnpm run validate-live-demo-links

# Validate Deploy to Cloudflare buttons
pnpm run validate-d2c-buttons
```

### Deployment Commands

```bash
# Deploy all templates
pnpm run deploy

# Deploy live demos
pnpm run deploy-live-demos

# Upload templates to Cloudflare Templates API
pnpm run upload
```

## Architecture Overview

### Template Structure

Each template follows a consistent structure:

- `package.json` with required Cloudflare metadata
- `wrangler.json` or `wrangler.jsonc` (not `.toml`)
- `README.md` with Deploy to Cloudflare button
- TypeScript configuration
- Worker source code typically in `src/`

### Custom CLI (`cli/`)

The repository includes a sophisticated CLI tool that:

- **Validates** template configuration and structure
- **Uploads** templates to Cloudflare's Templates API
- **Deploys** live demos for each template
- **Manages** dependencies across all templates
- **Enforces** consistent standards via linting

Key CLI commands used internally:

- `templates lint` - Validate template compliance
- `templates upload` - Upload to Templates API
- `templates deploy-live-demos` - Deploy previews

### Build System

Uses Turbo for coordinated builds across templates:

- Parallel building of multiple templates
- Caching for improved performance
- Coordinated TypeScript compilation

## Template Requirements and Best Practices

### What Are Templates?

Templates are high-quality, tangible use-case driven examples of applications built using Cloudflare Workers. They should be practical applications demonstrating technology, not sandboxed environments.

### Package.json Requirements

Templates must include:

- `name` ending in `-template` (matching directory)
- `description` (one-line summary, keep brief)
- `deploy` script for Wrangler deployment
- `cloudflare` metadata object with:
  - `label` (Title Case name for dashboard display)
  - `products` (max 3 Cloudflare products, focus on unique ones)
  - `categories` (from: "starter", "storage", "ai")
  - `preview_image_url` (16:9 aspect ratio, provided by Growth team)
  - `publish: true` (only if template should appear in dashboard)

### Visual and Content Requirements

- **Frontend Component**: Must have visual UI that clearly demonstrates functionality
- **Preview Image**: Required 16:9 aspect ratio screenshot (minimum 500px width)
- **README.md**: Must include:
  - Deploy to Cloudflare button
  - Getting started section with local development instructions
  - Dashboard content section wrapped in `<!-- dash-content-start -->` and `<!-- dash-content-end -->`
  - Screenshot of live application
  - Description of products used with documentation links

### Technical Requirements

- **Package-lock.json**: Required for 80% faster module resolution
- **Tests**: Mandatory minimum of 5 tests using vitest
- **Worker Binding**: Every template must use at least one Worker binding
- **API Router**: Use Hono unless project specifically needs different router
- **Top-level Environment**: Bindings must be top-level for Deploy to Cloudflare compatibility
- **Single Worker**: Cannot be monorepo, must be single Worker application

### Wrangler Configuration

- Must use `wrangler.json` or `wrangler.jsonc` format (not `.toml`)
- Latest compatibility date
- Observability and Smart Placement enabled by default
- Source maps enabled for development

### Code Standards and Best Practices

- **TypeScript**: Required for all templates
- **Framework Recommendations**: React + Vite, Next.js, React Router v7, or Astro
- **Latest Versions**: Use latest Wrangler and compatibility date
- **Comments**: Include educational comments explaining Cloudflare-specific features
- **Workers Assets**: Use Workers Assets for frontend instead of Pages
- **Naming**: Focus on use case, not underlying tech (e.g., "Astro AI Chat Bot" not "DO Template")
- **No Experimental Features**: Use stable, current best practices

### Secrets and Environment Variables

- List all required secrets in README.md
- Include where users can find appropriate values

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudflare/templates](https://github.com/cloudflare/templates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
