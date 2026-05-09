---
trigger: always_on
description: Browser-based tool that extracts design systems from Figma files and live websites, then transforms them into structured, LLM-optimised context bundles (layout.md) that enable AI coding agents to produce on-brand UI code consistently.
---

# Layout

## Overview

Browser-based tool that extracts design systems from Figma files and live websites, then transforms them into structured, LLM-optimised context bundles (layout.md) that enable AI coding agents to produce on-brand UI code consistently.

**Tagline:** The compiler between design systems and AI coding agents.

**Phase 1 scope:** Internal tool - no auth, no database, localStorage persistence only.

**Target users:** Developers using Claude Code, Cursor, GitHub Copilot, Windsurf to build UI.

## All Repos

| Repo | GitHub | Local Path | Visibility |
|---|---|---|---|
| **Web app** (this repo) | `uselayout/app` | `/Users/matt/Cursor Projects/Superduper AI Studio` | Public |
| **CLI + MCP server** | `uselayout/cli` | `/Users/matt/Cursor Projects/superduperui-context` | Public (MIT) |
| **Figma plugin** | `uselayout/figma` | `/Users/matt/Cursor Projects/Layout Figma Plugin/superduperui-figma` | Private |

- **CLI** (`@layoutdesign/context` on npm): Commands: `init`, `serve`, `install`, `doctor`, `list`, `use`, `import`. 12 MCP tools including `get_design_system`, `preview`, `push_to_figma`, `check_setup`. If the CLI needs new features, add them in that repo.
- **Figma plugin**: Native Figma plugin for token sync, component push, design system management.
- This repo provides the **backend API endpoints** the CLI calls (e.g. `/api/export/pull`).
- DO NOT duplicate CLI/MCP code here — it lives in `uselayout/cli`.

## Tech Stack

- **Frontend:** Next.js 15 (App Router), React 19, TypeScript strict mode, Tailwind CSS v4
- **UI:** shadcn/ui components, custom Studio design system
- **State:** Zustand with localStorage persistence
- **Editor:** @monaco-editor/react (markdown mode, custom dark theme)
- **AI:** @anthropic-ai/sdk - Claude Sonnet 4.6 (default), Claude Opus 4.7 (premium), Haiku 4.5 (simple edits). Models managed via admin panel (DB-driven).
- **Extraction:** Playwright (website), Figma REST API (Figma files)
- **Export:** JSZip for bundle generation
- **Validation:** Zod for all API inputs

## Commands

```bash
npm run dev          # Development server (http://localhost:3000)
npm run build        # Production build
npm run typecheck    # TypeScript type check (must pass)
npm run lint         # ESLint (must pass)
```

## Ralph

```bash
./scripts/ralph/ralph.sh        # Run autonomous build (42 iterations)
./scripts/ralph/ralph.sh 50     # Custom iteration count
```

Check story status:
```bash
cat scripts/ralph/prd.json | jq '.userStories[] | {id, title, passes}'
cat scripts/ralph/prd.json | jq '[.userStories[] | select(.passes == false)][0]'
```

## Project Structure

```
app/
  page.tsx                      # Landing page (URL input, AI Kit row)
  layout.tsx                    # Root layout with Geist fonts
  globals.css                   # Design tokens + Tailwind config
  studio/
    [id]/
      page.tsx                  # Two-panel Studio (Editor/Explore toggle)
      loading.tsx               # Loading skeleton
  (dashboard)/
    [org]/
      page.tsx                  # Projects list
      settings/                 # Org settings (API keys, billing, members, webhooks)
  api/
    extract/figma/route.ts      # Figma extraction → SSE stream
    extract/website/route.ts    # Website extraction → SSE stream
    generate/layout-md/route.ts # Claude layout.md synthesis → stream
    generate/explore/route.ts   # Explorer AI generation → stream
    export/bundle/route.ts      # ZIP bundle generation
    webhooks/figma/route.ts     # Figma webhook receiver
    health/completeness/route.ts # layout.md quality analysis
    organizations/[orgId]/      # Org-scoped API routes (components, members, etc.)
    transpile/route.ts          # TSX transpilation for preview

components/
  studio/
    StudioLayout.tsx            # Two-panel resize layout (source + editor/explore)
    SourcePanel.tsx             # Left panel (tokens, components, screenshots, quality, saved)
    EditorPanel.tsx             # Centre panel (Monaco)
    ExplorerCanvas.tsx          # AI-powered design exploration + validation
    ExplorerToolbar.tsx         # Explorer toolbar (prompts, image upload)
    VariantCard.tsx             # Individual variant display + actions
    CompletenessPanel.tsx       # layout.md quality score + suggestions
    ExtractionDiffModal.tsx     # Token/component diff on re-extraction
    ExtractionProgress.tsx      # Full-screen progress overlay
    ExportModal.tsx             # Export format selection + download
    PromoteToLibraryModal.tsx   # Save variant to component library
  shared/
    TopBar.tsx                  # Studio top bar
  ui/
    ConfirmModal.tsx            # Reusable confirm dialog
    # (other shadcn/ui components)

lib/
  figma/
    client.ts                   # Rate-limited Figma API fetch wrapper
    extractor.ts                # Extraction orchestrator
    parsers/
      styles.ts                 # Colour, text, effect styles
      components.ts             # Component inventory
  website/
    extractor.ts                # Playwright orchestrator

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uselayout/app](https://github.com/uselayout/app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
