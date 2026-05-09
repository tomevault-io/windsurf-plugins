---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Vibe AI Infra is a static website hub that tracks progress across multiple Nano AI Infrastructure projects. It's built with Vite + React + TypeScript using a minimal dark theme.

## Development Commands

```bash
# Install dependencies
npm install

# Start development server (http://localhost:5173)
npm run dev

# Type check without building (catches TypeScript errors)
npm run check

# Build for production
npm run build

# Run full CI pipeline (type check + build)
npm run ci

# Preview production build locally
npm run preview

# Render Mermaid diagrams to SVG
npm run render-mermaid

# Push with validation (RECOMMENDED over git push)
npm run push
```

## Pre-Commit and Pre-Push Workflow

**IMPORTANT: Always run validation before pushing to GitHub.**

### Why This Matters

- `npm run dev` does NOT run full TypeScript type checking
- TypeScript errors (unused variables, type mismatches) only appear during `astro check` or build
- GitHub Actions will fail if these errors aren't caught locally
- Broken deployments waste time and block the site from updating

### Required Validation Before Pushing

**Before committing or pushing, ALWAYS run:**

```bash
npm run ci
```

This runs:
1. `npm run check` - Full TypeScript type checking
2. `npm run build` - Production build verification

### Safe Push Workflow

**Option 1: Use npm run push (RECOMMENDED)**

```bash
npm run push
```

This automatically runs pre-push validation before pushing. If validation fails, the push is aborted.

**Option 2: Manual validation**

```bash
npm run ci && git push
```

**NEVER use `git push` directly without running `npm run ci` first.**

### What Gets Checked

The validation catches:
- TypeScript type errors (e.g., Date arithmetic operations)
- Unused variables and imports
- Astro component type mismatches
- Build failures due to syntax or dependency issues

### Example Failure

```typescript
// ❌ WRONG - Date arithmetic fails TypeScript check
entries.sort((a, b) => b.data.date - a.data.date);

// ✅ CORRECT - Use .getTime() for numeric comparison
entries.sort((a, b) => b.data.date.getTime() - a.data.date.getTime());
```

This error only appears in `npm run check`, not in `npm run dev`.

## Architecture

### Core Structure

- `src/projects.ts` - Single source of truth for all project data. Contains:
  - `projects` array: All 13 projects with status, goals, descriptions
  - `categories` array: The 5 top-level categories (Silicon, Virt, Compiler, Framework, Agent)
  - `statusOrder`: Controls sorting priority (Done > WIP > TBD)

- `src/App.tsx` - Main component that:
  - Computes statistics (done/wip/tbd counts)
  - Groups and sorts projects by category
  - Renders project cards with GitHub links

- `src/App.css` - Minimal dark theme styling matching GitHub/Vercel aesthetic

### Data Flow

1. `projects.ts` exports `projects` array as single source of truth
2. `App.tsx` filters projects by category and sorts by status
3. Component renders stats, category sections, and project cards

## Adding/Updating Projects

Edit `src/projects.ts` directly:

```typescript
{
  id: 'unique-id',
  name: 'Project Name',
  category: 'Silicon', // Must be one of: Silicon, Virt, Compiler, Framework, Agent
  status: 'WIP',       // 'TBD' | 'WIP' | 'Done'
  github: 'repo-name', // Optional - links to github.com/lastweek/repo-name
  description: 'Brief description of the project',
  goals: [
    'Specific goal 1',
    'Specific goal 2',
  ],
}
```

## GitHub Integration

- Set `base` path in `vite.config.ts` to match your repo name
- `github: 'repo-name'` in project data adds a GitHub icon linking to `github.com/lastweek/repo-name`
- No automation - status updates are manual via editing `src/projects.ts`

## Styling Conventions

- **Overall theme**: Light theme with soft gradients
- **Code blocks**: GitHub Light theme colors
  - Keywords: `#d73a49` (red)
  - Strings: `#032f62` (dark blue)
  - Comments: `#6a737d` (gray, italic)
  - Functions: `#6f42c1` (purple)
  - Numbers: `#005cc5` (blue)
  - Background: `#ffffff` (white)
  - Border: `#e1e4e8`
- **Status colors**:
  - Done: green `#3fb950`
  - WIP: yellow `#d29922`
  - TBD: gray `#8b949e`
- **Accent**: blue `#58a6ff` for links
- **Font**: SF Mono, Menlo, Monaco, Consolas for code
- **Border**: `rgba(255,255,255,0.1)` for subtle separation

## TIL (Today I Learned) Creation Workflow

When creating TIL entries, follow this process:

### Input
- User may provide **multiple topics** learned in a single day
- Content can be raw notes, links, code snippets, explanations, etc.

### Processing
1. Parse and deeply understand all provided content
2. Identify distinct topics/themes within the material

### When to Merge vs Separate Pages

**Use judgment based on topic relatedness:**

**Merge into ONE page when topics are:**
- About the same technology/project (e.g., both about SGLang)
- Conceptually related (e.g., native support + fallback mechanism)
- Part of a learning progression (e.g., basic concept + advanced application)
- Fewer than 3 distinct topics that tell a coherent story

**Create SEPARATE pages when topics are:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lastweek/Vibe-AI-Infra](https://github.com/lastweek/Vibe-AI-Infra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
