---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Smart Drawio Next is a Next.js application that generates editable Draw.io diagrams from natural language descriptions or reference images using LLM APIs. Users describe what they want (e.g., "a Transformer architecture diagram") and the system streams back Draw.io XML that renders in an embedded draw.io iframe.

## Technology Stack

- **Framework**: Next.js 16 with App Router + React 19
- **UI**: Tailwind CSS v4 with CSS custom properties for light/dark theming (`globals.css`)
- **Canvas**: Embedded draw.io iframe (dynamically imported, SSR-disabled)
- **Editor**: Monaco Editor (`@monaco-editor/react`)
- **LLM Integration**: OpenAI/Anthropic-compatible APIs with SSE streaming
- **Validation**: Zod (`lib/schemas.js`)
- **State**: Client-side localStorage via `lib/storage.js` wrapper
- **Testing**: Vitest (unit) + Playwright (E2E)

## Development Commands

```bash
pnpm install                    # Install dependencies
pnpm dev                        # Start dev server (webpack mode)
pnpm build                      # Production build
pnpm start                      # Start production server
pnpm lint                       # Run ESLint
pnpm test                       # Run unit tests (vitest, watch mode)
pnpm test -- run                # Run unit tests once (no watch)
pnpm test -- tests/lib/theme-engine.test.js   # Run a single test file
pnpm test:coverage              # Run tests with v8 coverage
pnpm test:ui                    # Run tests with Vitest UI
pnpm test:e2e                   # Run E2E tests (Playwright, needs dev server)
```

**Requirements**: Node.js >= 18.18, pnpm

## Architecture

### Core Generation Flow

```
User Input (Chat.jsx)
  -> app/page.js orchestrates state
    -> useGenerationWorkflow hook
      -> POST /api/generate (SSE streaming)
        -> lib/generate-route-utils.js (validation, message building)
          -> lib/llm-client.js (OpenAI/Anthropic adapter)
            -> Stream chunks back via SSE
              -> lib/drawio-code-utils.js (parse, sanitize, post-process)
                -> lib/theme-postprocess.js (auto-apply tricks per theme)
                  -> CodeEditor + DrawioCanvas update
```

### State Management

All application state lives in `app/page.js` and flows down as props. No global state library. Complex logic is extracted into three custom hooks:

| Hook | Location | Purpose |
|------|----------|---------|
| `useGenerationWorkflow` | `lib/hooks/use-generation-workflow.js` | Generation lifecycle: streaming, continuation, optimization, error handling, blueprint phases |
| `useToolsPanel` | `lib/hooks/use-tools-panel.js` | Style preset toggles, drawing tricks, text tools, style packs |
| `useXmlHistory` | `lib/hooks/use-xml-history.js` | Undo/redo stack for XML snapshots (max 20 entries) |

Pattern for adding new state: add to `app/page.js`, pass handlers/state as props, use `useEffect` for localStorage sync.

### Tools System

The tools system (`lib/tool-registry.js`) aggregates four categories of XML post-processing tools, all following the same pattern: take XML in, return `{ xml, error?, stats? }`.

| Category | Module | What it does |
|----------|--------|-------------|
| Style Presets | `lib/style-presets.js` | Toggle-based effects (shadow, gradient, rounded, glass) |
| Text Style Tools | `lib/text-style-tools.js` | Font/text transformations |
| Style Packs | `lib/style-packs.js` | Multi-property style bundles |
| Drawing Tricks | `lib/drawing-tricks.js` | Structural transforms (grid snap, smart arrows, orthogonal routing, label backgrounds) |

### Theme System

Two separate theme systems:

1. **UI Theme** (light/dark/system): CSS custom properties in `globals.css`, toggled via `data-theme` attribute. Managed by `lib/theme-mode.js`.
2. **Diagram Theme** (10 color palettes): `lib/themes/` directory. Each theme defines a `colorPalette` with semantic tokens. `lib/theme-engine.js` handles XML style parsing (`parseStyle`/`stringifyStyle`) and color remapping. `lib/theme-postprocess.js` auto-applies certain drawing tricks per theme (e.g., research theme gets orthogonal routing + smart arrows).

### Storage

Uses `lib/storage.js` wrapper with in-memory Map fallback for SSR/privacy mode. Keys prefixed with `smart-drawio-*`:

| Key | Purpose |
|-----|---------|
| `smart-drawio-configs` | Array of LLM configs |
| `smart-drawio-active-config` | Current config ID |
| `smart-drawio-history` | Last 20 generations |
| `smart-drawio-diagram-theme` | Current diagram theme |
| `smart-drawio-style-presets` | Active style presets |
| `smart-drawio-theme-mode` | UI theme (light/dark/system) |
| `smart-drawio-active-panel` | Currently open sidebar panel |

Legacy keys (`smart-excalidraw-*`) are auto-migrated on first load via `ensureMigration()`.

### API Routes

| Route | Method | Purpose |
|-------|--------|---------|
| `/api/generate` | POST | Main generation endpoint (SSE streaming), rate-limited |
| `/api/models` | GET | List available models from configured provider |
| `/api/configs` | POST | Test LLM connection |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yunshenwuchuxun/smart-drawio-next](https://github.com/yunshenwuchuxun/smart-drawio-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
