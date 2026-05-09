---
trigger: always_on
description: **pnpm monorepo** with three workspace packages:
---

# CLAUDE.md

## Project Overview

**pnpm monorepo** with three workspace packages:

1. **`@endorhq/capsule-web`** (`packages/web`) — SvelteKit web app that visualizes conversation logs from AI coding agents (Claude Code, Codex, Copilot, Gemini). Parses different log formats into a unified timeline and renders them in a terminal-inspired dark UI. Builds with adapter-node (default, for CLI consumption) or adapter-cloudflare (for production deploy).
2. **`@endorhq/capsule-shared`** (`packages/shared`) — Shared parsers and types consumed by both the web app and the CLI.
3. **`@endorhq/capsule`** (`packages/cli`) — CLI tool (`capsule`) with three subcommands: `share` (publish to GitHub Gist), `export` (save to local file), and `serve` (start a local web viewer).

The root (`./`) is not a workspace package — it holds monorepo tooling only (biome, lefthook).

## Tech Stack

- **SvelteKit 2** + **Svelte 5** (runes: `$state`, `$derived`, `$props`)
- **TypeScript** (strict mode)
- **Tailwind CSS v4** (new `@theme` syntax, `@tailwindcss/vite` plugin)
- **Vite 7**
- **pnpm** package manager (monorepo via `pnpm-workspace.yaml`)
- **tsup** (CLI bundling) + **tsx** (CLI dev)
- **@clack/prompts** + **picocolors** (CLI interactive UI)
- **@sveltejs/adapter-node** (default build, for `capsule serve`)
- **@sveltejs/adapter-cloudflare** (opt-in via `ADAPTER=cloudflare`, for production)

## Commands

### Web App (`packages/web`)

- `pnpm dev` — Start dev server (proxied from root)
- `pnpm build` — Build with adapter-node + `PUBLIC_DISTRIBUTION=local` (proxied from root)
- `pnpm -C packages/web build:cloudflare` — Build with adapter-cloudflare for production
- `pnpm -C packages/web check` — Type-check (svelte-kit sync + svelte-check)
- `pnpm -C packages/web preview` — Preview production build

### CLI (`packages/cli`)

- `capsule share [file]` — Publish a session to GitHub Gist
- `capsule export [file]` — Save an anonymized session to a local file
- `capsule serve [--port N]` — Start a local web viewer (default port 3123)
- `capsule --help` — Print usage
- `pnpm -C packages/cli dev` — Run CLI via tsx (use `npx tsx src/index.ts <command>` for subcommands)
- `pnpm -C packages/cli build` — Bundle CLI to `dist/index.js`

### Root

- `pnpm dev` — Proxy to `pnpm -C packages/web dev`
- `pnpm build` — Proxy to `pnpm -C packages/web build`
- `pnpm lint` — Run biome lint
- `pnpm format` — Run biome format

## Architecture

### Monorepo Layout

```
.                          — Root: monorepo tooling only
wrangler.jsonc             — Cloudflare Workers config (at root for CF auto-detection)
packages/
  web/                     — @endorhq/capsule-web: SvelteKit app
    src/
      lib/
        features.ts        — Feature flag: isLocal / distribution
        components/        — UI components
          viewer/          — Session viewer (MessageThread, FilterBar, entries, panels)
        parsers/           — Thin re-exports from @endorhq/capsule-shared
        services/          — Browser storage (OPFS > IndexedDB > memory)
        state/             — Svelte 5 rune-based state
        types/             — Thin re-exports from @endorhq/capsule-shared
      routes/              — SvelteKit routes (+page.svelte, +layout.svelte, layout.css)
    svelte.config.js       — Conditional adapter (node default, cloudflare opt-in)
    vite.config.ts         — Tailwind + SvelteKit plugins
    .env                   — PUBLIC_DISTRIBUTION=public (default for dev)
  shared/                  — @endorhq/capsule-shared: parsers + types
    src/
      parsers/             — claude.ts, codex.ts, copilot.ts, gemini.ts, detect.ts, index.ts
      types/               — timeline.ts (ParsedSession, TimelineEntry, etc.)
  cli/                     — @endorhq/capsule: CLI binary
    src/
      index.ts             — Subcommand router (share, export, serve)
      commands/
        share.ts           — capsule share: gh auth → session → anonymize → gist
        export.ts          — capsule export: session → anonymize → save file
        serve.ts           — capsule serve: import handler → http server
      flows/
        session.ts         — Shared: file arg OR discovery → { content, format }
        anonymize-prompt.ts — Shared: multiselect with "Select all" + apply
      discovery.ts         — Find sessions on disk per agent
      anonymize.ts         — Format-aware anonymization transforms
      publish.ts           — Gist creation + file save via gh CLI
```

### Web App Layout

Three-panel layout: sidebar (session list + upload) | center (message timeline + filter) | right (session metadata panel).

### Data Flow (Web App)

1. User uploads a `.jsonl`/`.json` file via UploadZone or drag-drop
2. `storage.svelte.ts` stores the file and detects the agent format
3. On session select, `sessions.svelte.ts` reads raw content, calls `parseSession()`
4. Parser returns a `ParsedSession` with a flat `TimelineEntry[]` timeline
5. `SessionViewer.svelte` renders the timeline with filtering, grouping tool calls into nested blocks

### CLI Subcommands

**`capsule share [file]`** — Check `gh` auth first (fail early), resolve session (file arg or interactive discovery), prompt anonymization, prompt gist visibility, publish gist, print viewer URL.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [endorhq/capsule](https://github.com/endorhq/capsule) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
