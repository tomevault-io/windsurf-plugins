---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a **Mintlify-based documentation monorepo** for Auth0. It contains multiple independent documentation sites:

- **`main/`** - Primary Auth0 documentation (https://auth0.com/docs)
- **`auth4genai/`** - Auth0 for AI Agents documentation (https://auth0.com/ai/docs)
- **`ui/`** - Shared React/Vite component library used across documentation sites
- **`universal-components/`** - Shared React/Vite component library used for interactive components documentations

Each documentation site (`main`, `auth4genai`) operates independently with its own `docs.json` Mintlify configuration file.

## Common Commands

### Documentation Development

All documentation commands use the Mintlify CLI (`mint`). You must navigate to the specific documentation folder (where `docs.json` exists) before running these commands.

```bash
# Install Mintlify CLI globally (prerequisite: Node.js v19+)
npm i -g mint

# Start local development server (from main/ or auth4genai/)
cd main  # or cd auth4genai
mint dev  # Opens at http://localhost:3000

# Use custom port
mint dev --port 3333

# Update Mintlify CLI
mint update
# or
npm i -g mint@latest

# Find broken links
mint broken-links

# Check accessibility issues
mint a11y
```

> **VPN Note:** When running `mint dev` for the first time, disable your VPN to allow framework download. You can re-enable it after the initial setup.

### UI Component Library

The shared UI library is in `/ui` and must be built before changes are visible in documentation sites.

```bash
cd ui

# Install dependencies
npm install  # or pnpm install

# Development server (Vite)
npm run dev  # or pnpm dev

# Build library (required after changes)
npm run build  # or pnpm build
# Output: auth0-docs-ui-{version}.umd.js and .css in /ui directory

# Lint
npm run lint

# Format
npm run format
```

## Architecture

### Monorepo Structure

This is **not a managed monorepo** (no Lerna, pnpm workspaces, etc.). Each folder is independent:

- Documentation sites (`main/`, `auth4genai/`) contain their own content and configuration
- Shared UI library (`ui/`) is built separately and included in documentation sites
- No package manager workspace configuration at root level

### Documentation Organization

**Content Structure:**

- `.mdx` and `.md` files for documentation pages
- YAML frontmatter for metadata (title, description)
- `docs.json` defines navigation structure and Mintlify configuration

**Reusable Components:**

- `/snippets` directories contain reusable `.mdx` and `.jsx` components
- Import snippets into documentation pages to avoid duplication
- Commonly used for multi-language code examples in quickstart guides

**Code Block Convention:**

`````markdown
````[language] [filename] wrap lines highlight={lines}
Example: ```typescript ./src/auth0/app wrap lines highlight={1,7-10}
````
`````

`````

**Localization:**

- Main docs support French Canadian (`main/docs/fr-ca/`) and Japanese (`main/docs/ja-jp/`)

### UI Component Library Architecture

**Technology Stack:**

- React 19 + TypeScript
- Vite 7 for building
- TailwindCSS 4 for styling
- Radix UI + shadcn/ui for component primitives
- MobX 6 for state management

**State Management:**

- MobX stores pattern with `RootStore` as central container
- Key stores: `SessionStore`, `ClientStore`, `TenantStore`, `ResourceServerStore`, `VariableStore`
- Components use MobX `observer` wrapper for reactivity

**Build Output:**

- UMD bundle: `auth0-docs-ui-{version}.umd.js`
- CSS: `auth0-docs-ui-{version}.css`
- Exposed as `window.Auth0DocsUI` in browser
- Exports: components, stores, and MobX utilities

**Path Aliases:**

- `@/*` maps to `/ui/src/*` for clean imports

### Theme Configuration

**Main Docs (`main/docs.json`):**

- Theme: "aspen"
- Colors: Black primary (#000)
- Breadcrumb navigation style
- Traditional layout

**Auth4GenAI Docs (`auth4genai/docs.json`):**

- Theme: "mint"
- Colors: Purple primary (#6742D5)
- Dark mode by default
- Gradient backgrounds
- IDE/MCP integration support (contextual options: vscode, cursor, mcp)

## Deployment

- **Automatic deployment** via Mintlify's GitHub App integration
- Changes to default branch are automatically deployed to production
- No manual deployment commands or GitHub Actions workflows needed
- Focus on committing to the correct branch

## Key Workflow Patterns

### Making Documentation Changes

1. Navigate to the appropriate docs folder (`main/` or `auth4genai/`)
2. Edit `.mdx` or `.md` files
3. Run `mint dev` to preview changes locally
4. Commit and push to trigger automatic deployment

### Creating Reusable Components

1. Add component to `/snippets` directory (`.mdx` or `.jsx`)
2. Import into documentation pages as needed
3. Useful for code examples shared across multiple pages

### Modifying UI Components

1. Make changes in `/ui/src/components/`
2. Run `npm run build` in `/ui` directory
3. Test in documentation site by running `mint dev`
4. Commit both UI changes and built files

### Working with Current Branch

- Current branch: `feat/auth-for-mcp-new-docs`
- Focus: Model Context Protocol (MCP) documentation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [auth0/docs-v2](https://github.com/auth0/docs-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
