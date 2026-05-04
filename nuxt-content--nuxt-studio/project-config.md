---
trigger: always_on
description: Nuxt Studio is an open-source, self-hostable Nuxt module that enables visual content editing in production for [Nuxt Content](https://content.nuxt.com) websites. Originally a premium hosted platform, it's now a free MIT-licensed module that runs entirely on your own infrastructure.
---

# Nuxt Studio

## Overview

Nuxt Studio is an open-source, self-hostable Nuxt module that enables visual content editing in production for [Nuxt Content](https://content.nuxt.com) websites. Originally a premium hosted platform, it's now a free MIT-licensed module that runs entirely on your own infrastructure.

**Key Concept**: This module adds a full-featured CMS editor directly into your Nuxt Content application, allowing non-technical users to edit content and commit changes to Git without needing local development tools.

## Core Features

- **Visual Editors**: TipTap-based Notion-like editor for Markdown with MDC component support, form-based editor for YAML/JSON
- **Code Editor**: Monaco editor with syntax highlighting for direct file editing
- **Real-time Preview**: See changes instantly on production website
- **Git Integration**: Commits directly to GitHub/GitLab repositories
- **Multi-provider Auth**: GitHub, GitLab, Google OAuth, or custom authentication
- **Media Management**: Visual media library with drag-and-drop support
- **Development Mode**: Local filesystem sync for development
- **Production Mode**: OAuth + Git publishing for deployed sites
- **i18n Support**: 25 languages built-in

## Architecture

### Project Structure

```
studio/
├── src/
│   ├── app/           # Vue app for the Studio editor interface
│   │   ├── src/       # Vue components, composables, utils
│   │   └── service-worker.ts
│   └── module/        # Nuxt module code
│       └── src/
│           ├── runtime/  # Runtime server routes & plugins
│           └── module.ts # Main module definition
├── playground/        # Development examples
│   ├── docus/         # Full-featured example
│   └── minimal/       # Minimal example
├── docs/              # Documentation site (also a Nuxt Content app)
```

### Two Operating Modes

1. **Development Mode** (default in dev)
   - Direct filesystem access via server routes
   - No auth required
   - Changes sync immediately to local files
   - No Git operations

2. **Production Mode** (default in prod)
   - OAuth authentication required
   - Git provider integration for commits
   - Changes pushed to repository
   - Triggers CI/CD pipeline

## Key Technologies

- **Nuxt 3**: Core framework
- **Nuxt Content**: Content management layer (peer dependency)
- **@nuxtjs/mdc**: MDC (Markdown Components) parsing and rendering
- **TipTap**: Visual WYSIWYG editor
- **Monaco Editor**: Code editor
- **Vue Router**: SPA routing inside Studio
- **IndexedDB**: Client-side draft storage via `idb-keyval`
- **Service Worker**: Offline support and caching
- **Shiki**: Syntax highlighting
- **Zod**: Schema validation for forms

## Configuration

### Basic Setup

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxt/content', 'nuxt-studio'],

  studio: {
    route: '/_studio',  // Admin route

    // Git repository config (required for production)
    repository: {
      provider: 'github',  // 'github' | 'gitlab'
      owner: 'username',
      repo: 'repo-name',
      branch: 'main',
      rootDir: '',         // For monorepos
      private: true        // Request private repo access
    },

    // i18n
    i18n: {
      defaultLocale: 'en'  // 25 languages available
    },

    // Component filtering
    meta: {
      components: {
        include: ['Content*'],  // Whitelist
        exclude: ['Hidden*']    // Blacklist
      }
    }
  }
})
```

### Environment Variables

**GitHub OAuth + GitHub Git Provider**:
```bash
STUDIO_GITHUB_CLIENT_ID=xxx
STUDIO_GITHUB_CLIENT_SECRET=xxx
STUDIO_GITHUB_MODERATORS=email1@example.com,email2@example.com  # Optional
```

**GitLab OAuth + GitLab Git Provider**:
```bash
STUDIO_GITLAB_APPLICATION_ID=xxx
STUDIO_GITLAB_APPLICATION_SECRET=xxx
STUDIO_GITLAB_MODERATORS=email1@example.com,email2@example.com  # Optional
```

**Google OAuth** (requires PAT):
```bash
STUDIO_GOOGLE_CLIENT_ID=xxx
STUDIO_GOOGLE_CLIENT_SECRET=xxx
STUDIO_GOOGLE_MODERATORS=email1@example.com,email2@example.com  # Required!
STUDIO_GITHUB_TOKEN=xxx  # or STUDIO_GITLAB_TOKEN
```

**Custom Auth** (requires PAT):
```bash
STUDIO_GITHUB_TOKEN=xxx  # or STUDIO_GITLAB_TOKEN
```

## Authentication vs Git Providers

**Important distinction**:
- **Auth Providers**: Control who can login (GitHub OAuth, GitLab OAuth, Google OAuth, Custom)
- **Git Providers**: Control where content is committed (GitHub, GitLab)

You can mix and match: e.g., Google OAuth for auth + GitHub for Git operations (requires PAT).

## Development Workflow

### Setup

```bash
# Install dependencies
pnpm install

# Generate type stubs
pnpm dev:prepare

# Build app and service worker
pnpm prepack
```

### Running Locally

```bash
# Terminal 1: Start the Vue app dev server
pnpm dev:app  # Runs on :5151

# Terminal 2: Start the Nuxt playground
pnpm dev      # Runs on :3000, points to :5151 for Studio app
```

### Testing

```bash
pnpm verify      # Runs all checks
pnpm test        # Vitest tests
pnpm typecheck   # Type checking
pnpm lint        # ESLint
```

## Key Concepts

### File types


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nuxt-content/nuxt-studio](https://github.com/nuxt-content/nuxt-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
