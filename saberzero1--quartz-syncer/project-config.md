---
trigger: always_on
description: Obsidian Community Plugin. Publishes Obsidian notes to [Quartz](https://quartz.jzhao.xyz/) static sites via Git over HTTPS.
---

# Quartz Syncer

Obsidian Community Plugin. Publishes Obsidian notes to [Quartz](https://quartz.jzhao.xyz/) static sites via Git over HTTPS.

## Build

- Package manager: npm
- Bundler: esbuild (`node esbuild.config.mjs production`)
- Unit tests: Vitest (`npx vitest run`)
- E2E tests: WebdriverIO
- Integration tests: Playwright
- Type check: `npx tsc --noEmit`
- isomorphic-git fork: `saberzero1/isomorphic-git`. The `package.json` dependency MUST point to `https://github.com/saberzero1/isomorphic-git.git` before committing — never `file:../isomorphic-git`.

## Architecture

### Two-tier platform split

**Core (desktop + mobile):** Publish, sync, delete, status, diff, cache, mark, background compilation. Uses `BundledGitBackend` (isomorphic-git fork) for all Git I/O — no shell commands.

**Management (desktop only):** Quartz config, plugins, upgrades, templates, auto-publish, local preview. Uses `ProcessRunner` → `QuartzRunner` for `npx quartz` commands. Requires local Quartz checkout + Node.js ≥18.

### Key modules

- `BundledGitBackend` — primary Git transport via isomorphic-git. Works everywhere.
- `QuartzFileSource` — interface for reading/writing Quartz repo files. Two implementations: `RemoteFileSource` (Git remote) and `LocalFileSource` (local disk).
- `Publisher` — orchestrates publish/delete/status via `BundledGitBackend` + `DataStore` + `PathMapper`.
- `PublishStatusManager` — categorizes files into unpublished/changed/published/deleted/media.
- `MediaLinkResolver` — tracks which media files are linked by published notes.
- `SyncerPageCompiler` — compilation pipeline: frontmatter → markdown → integration adapters.
- `BackgroundEngine` — watches vault changes, queues compilation, auto-publish timer.
- `ProcessRunner` — desktop-only system command execution with circuit breaker. Timeout sentinel: `-1` = no timeout.
- `QuartzRunner` — wraps `npx quartz` subcommands. `serve()` bypasses `ProcessRunner` singleton to avoid pending process kills.
- `NodeDetector` — checks Node.js ≥18 availability.

### Quartz Hub

Desktop-only modal (`src/views/QuartzHub/`) for local Quartz repository management. Accessible from command palette (`quartz-syncer:open-hub`), settings ("Open Quartz Hub" button), and operability facade (`act('hub.open')`). Tab-based layout:

- **Overview** — repo status (path, Quartz version, binaries, serve state) + action buttons (Preview, Build, Update, Install deps, Plugins, Open folder)
- **Setup** — link existing local repo (path → validate → save) or clone from remote (URL → git clone → npm install → save)

Services: `QuartzHubService` (`src/services/QuartzHubService.ts`) handles status assembly, path validation, and preflight checks. `QuartzHubManager` (`src/operability/QuartzHubManager.ts`) is the singleton modal owner.

### Publication model

Publishing destination is explicit, never inferred. `publishTarget` (`"local" | "remote"`) selects it; `resolvePublishTarget()` in `src/publisher/PublishTargetResolver.ts` is the single source of truth and every publication surface must use it instead of testing `quartzRepoPath`/`gitRemoteUrl` directly. A requested-but-unconfigured destination resolves to `null` rather than silently falling back; the only automatic override is local-on-mobile, which reports `overridden: true`.

`quartzRepoPath` has a second, independent role: the local checkout used for Quartz site management (Hub, build, preview, plugins). Management code reads it directly regardless of `publishTarget`, and nothing may clear it to switch destinations.

Publish status is computed against one destination's tree, so `StatusCacheService` is keyed by `publishTargetIdentity()`. Results arriving for a stale destination are discarded.

`publish: true` in frontmatter marks a file as **publishable** — visible in the Publication Center. It does NOT auto-publish. The user selects which files to publish. ALL files in the Publication Center are selectable, including "Published" (already synced) files.

Media files linked by notes are pushed alongside them automatically. Orphaned media (unlinked) can be cleaned automatically via `autoCleanOrphanedMedia` setting.

### Publication Center

Uses persistent shell + `PublicationTree` class with keyed DOM row maps. State changes update checkbox properties and CSS classes in-place — no full DOM rebuilds. This preserves `checkbox.indeterminate`, scroll position, and input focus.

## CLI

22 commands registered via `registerCliHandler()` (Obsidian 1.12.2+ API). NOT `registerObsidianProtocolHandler` — that is for URL protocol handling, not CLI.

Commands are defined in `COMMAND_REGISTRY` in `src/cli/registerCliHandlers.ts`. The CLI itself is desktop-only — no `Platform.isDesktopApp` checks in handlers.

## Quartz v5

Quartz v5 uses `quartz.config.default.yaml` as the base configuration. `quartz.config.yaml` is optional — it contains user overrides only. If absent, Quartz falls back to the default. Do not assume `quartz.config.yaml` exists in a fresh Quartz repo.

The Quartz CLI is accessed via `npx quartz <command>`. Available commands: `create`, `upgrade`/`update`, `restore`, `sync`, `build`, `tui`, `plugin [subcommand]`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [saberzero1/quartz-syncer](https://github.com/saberzero1/quartz-syncer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
