---
trigger: always_on
description: This file provides guidance to AI coding agents (Claude Code, Codex, Cursor, etc.) when working with code in this repository.
---

# CLAUDE.md / AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Codex, Cursor, etc.) when working with code in this repository.

## Project Overview

Persona is a pnpm monorepo containing a themeable, pluggable streaming chat widget for websites. It consists of two publishable packages and three example applications.

**Packages:**
- `packages/widget` (`@runtypelabs/persona`) - The main chat widget library
- `packages/proxy` (`@runtypelabs/persona-proxy`) - Optional Hono-based proxy server

**Examples:**
- `examples/embedded-app` - Vite demo with vanilla JS (35+ demo pages)
- `examples/vercel-edge` - Node.js proxy for Vercel/Railway/Fly.io
- `examples/cloudflare-workers` - Edge proxy for Cloudflare Workers

## Requirements

- **Node.js** ≥20 (see `.nvmrc`)
- **pnpm** — managed via corepack (`corepack enable` then `corepack install`)

## Common Commands

```bash
# Development
pnpm dev                # Start proxy (port 43111) + widget demo (port 5173) concurrently
pnpm dev:widget         # Start widget demo only
pnpm dev:vercel         # Start vercel-edge proxy only
pnpm dev:cloudflare     # Start cloudflare-workers proxy only

# Build
pnpm build              # Build both widget and proxy packages
pnpm build:widget       # Build widget only
pnpm build:proxy        # Build proxy only

# Quality
pnpm lint               # Lint both packages
pnpm typecheck          # Type check both packages

# Testing (widget package only — run from repo root or packages/widget)
pnpm --filter @runtypelabs/persona test:run   # Run tests once
pnpm --filter @runtypelabs/persona test       # Run tests in watch mode
pnpm --filter @runtypelabs/persona test:ui    # Run tests with browser UI (http://localhost:51204)

# Releases
pnpm changeset          # Create a changeset for version tracking
pnpm changeset version  # Apply changesets to bump versions
pnpm release            # Publish to npm
```

## Changesets Requirement

All changes that affect published packages **must** include a changeset. This is enforced in CI — PRs without a changeset for affected packages will fail.

Create one by adding a markdown file in `.changeset/` (e.g. `.changeset/my-change-name.md`) with the following format:

```md
---
"@runtypelabs/persona": patch
---

Short description of the change for the changelog.
```

Use `patch` for bug fixes, `minor` for new features, and `major` for breaking changes. If both packages are affected, list both in the frontmatter.

**Requires a changeset:** Any change to `packages/widget` or `packages/proxy` (source, types, styles, dependencies).

**Does NOT need a changeset:** Changes to `examples/`, `docs/`, CI config, tooling, or `CLAUDE.md`.

## Architecture

### Widget Package (`packages/widget/src/`)

The widget uses a layered architecture:

1. **Entry Points**
   - `index.ts` - Public API exports
   - `install.ts` - Script tag installer for CDN usage
   - `runtime/init.ts` - Widget initialization (Shadow DOM setup)
   - `runtime/host-layout.ts` - Layout management for docked mode

2. **Core Layer**
   - `client.ts` - HTTP client handling SSE streaming, message dispatch, and API communication
   - `session.ts` - Message state management, injection methods, client session handling
   - `types.ts` - All TypeScript type definitions (~2900 lines; covers multi-modal content, agent loop config, tool config, theme types, request/response payloads)

3. **UI Layer** (`ui.ts` + `components/`)
   - `ui.ts` - Main UI controller, DOM rendering, event handling
   - `components/` - Modular UI components (launcher, panel, header, messages, feedback, artifacts, tool bubbles, reasoning, approval, suggestions, forms, composer)
   - `styles/widget.css` - Tailwind CSS with `tvw-` prefix for style scoping

4. **Utilities** (`utils/`)
   - `formatting.ts` - Stream parsers (JSON, XML, plain text)
   - `content.ts` - Multi-modal content handling (text, images, files)
   - `attachment-manager.ts` - File attachment handling
   - `actions.ts` - AI action parsing and handling
   - `component-parser.ts` / `component-middleware.ts` - Dynamic component system
   - `tokens.ts` / `theme.ts` - Design tokens and theme utilities
   - `virtual-scroller.ts` - Performance optimization for long message lists
   - `event-stream-*.ts` - SSE event buffering, capture, control, and storage

5. **Voice** (`voice/`)
   - `browser-voice-provider.ts` - Web Audio API voice input
   - `runtype-voice-provider.ts` - Runtype-hosted voice service
   - `voice-activity-detector.ts` - VAD implementation
   - `voice-factory.ts` - Provider factory

6. **Extensibility**
   - `plugins/` - Plugin registry system for custom functionality (`types.ts`, `registry.ts`)
   - `postprocessors.ts` - Markdown and directive postprocessors
   - `defaults.ts` - Default configuration with light/dark theme presets

### Key Patterns

**Streaming:** SSE-based with pluggable parsers. Uses `partial-json` for incomplete JSON chunks during streaming.

**Content Priority Chain:** When building API payloads, content resolves as: `contentParts > llmContent > rawContent > content`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [runtypelabs/persona](https://github.com/runtypelabs/persona) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
