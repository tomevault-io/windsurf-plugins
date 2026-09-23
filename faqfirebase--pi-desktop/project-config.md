---
trigger: always_on
description: An Electron desktop application that acts as a GUI frontend for the Pi coding agent. Currently in alpha; see the Version section below.
---

# Pi Desktop

An Electron desktop application that acts as a GUI frontend for the Pi coding agent. Currently in alpha; see the Version section below.

## Version

Project is currently in **Alpha**. APIs, IPC contracts, on-disk config formats, and packaged-app behavior may all change without notice. Do not rely on anything as stable.

- Never refer to alpha releases as production-ready
- Breaking changes are acceptable before 1.0.0
- Preserve forward migration paths whenever practical

## Architecture

### Stack

- **Electron** — Desktop shell with secure IPC
- **React 19** — UI framework
- **TypeScript** — Full type safety
- **Vite** — Build tooling via electron-vite
- **TailwindCSS v4** — Styling
- **Zustand** — State management
- **Pi RPC Mode** — JSONL-based subprocess communication

### Security

- `contextIsolation: true`
- `nodeIntegration: false`
- `sandbox: true`
- All IPC channels validated with typed contracts
- No renderer access to Node APIs
- Main-window navigation pinned to the packaged renderer; privileged IPC verifies the sender frame is the app renderer
- Per-workspace trust gate: an untrusted workspace's own `.pi-desktop/permission-rules.json` allow rules are ignored, and its HTML preview runs without scripts/network, until the user trusts the workspace
- Attachment reads limited to picked or in-workspace paths; session deletion confined to the Pi sessions dir; package specs validated before the Pi CLI runs

### Interface text

- One i18next default instance per process (`src/shared/i18n`), starting in English; language files live in `resources/locales/<code>/translation.json`
- Components call `useTranslation()` from `react-i18next`; other code imports `t` from `src/shared/i18n`. Text is built when called, never stored translated at module load
- `tEnglish` (also from `src/shared/i18n`) is used instead of `t` for logs, `appLog.*`, and the diagnostics report, which stay English regardless of the interface language
- Product names ("Pi Desktop", "Pi", "OMP") are never translation keys — they come from `agentEngineLabel()`, `councilAgentLabel()`, or a named constant, never from a language file
- `npm run lint` runs `i18next-cli lint` and `i18next-cli extract --ci --dry-run`, so hard-coded text and stale keys fail CI
- Code never decides behavior from a translated display string; it reads underlying values (failure codes, `kind`/`ToolKind` enums, error types) instead
- `t` from `useTranslation()` changes identity on every language switch. List it in the deps of a `useMemo`/`useCallback` that builds text, but never in the deps of a `useEffect` that does I/O or resets state (a re-run once discarded unsaved editor edits): keep the outcome as data and translate it at render (`utils/preview-load-error.ts`)
- `i18next-cli extract` finds keys only in `t('literal.key')` calls on an identifier named `t`, or through a key map declared in the same file. A translator passed to a helper must be a parameter named `t`; a key map imported from another file is invisible, and `removeUnusedKeys` deletes its keys. Share labels through a helper that calls `t` directly (`utils/process-status-label.ts`)

## Project Structure

Modules have colocated `*.test.ts` files; `resources/` has tests too. CI runs `npx tsx --test $(find src resources -name '*.test.ts')`.

```
src/
├── shared/                       # Code shared by main + renderer (pure, typed)
│   ├── ipc-contracts.ts          # Typed IPC channel definitions
│   ├── default-settings.ts       # Single source of truth for AppSettings defaults
│   ├── council-config.ts         # Council planning config, prompts, parsers
│   ├── models-config.ts          # Custom models.json validate/merge
│   ├── package-filter.ts         # Tokenized catalog search, shared main+renderer
│   ├── package-spec.ts           # Validate package specs before the Pi CLI runs
│   ├── version-compare.ts        # x.y.z-prerelease ordering (app and package update checks)
│   ├── path-compare.ts           # Platform-aware path equality (win32 case-fold); main+renderer
│   ├── folder-drop.ts            # Pure helpers for drag-drop folder → workspace
│   ├── untrusted-data.ts         # Wrap file/agent text as a labeled untrusted-data block
│   ├── agent-engine-label.ts     # Display names for the Pi/OMP engines (every surface reads this one map)
│   ├── product-name.ts           # "Pi Desktop" display name (a named constant, never a translation key)
│   ├── i18n/                     # i18next instance, t/tEnglish, bundled languages, OS-language resolver, pseudo-language, locale checks
│   ├── pi-command.ts             # Slash-command filtering
│   ├── fork-point.ts             # Fork/branch message helpers
│   ├── session-lineage.ts        # Cross-session lineage tree
│   ├── session-preview.ts        # First user message -> one-line row label
│   ├── sidebar-width.ts          # Bounds/resolution for the user-adjustable sidebar width
│   ├── workflow-control.ts       # Control eligibility for persisted workflow runs
│   └── theme/                    # Theme-file format, resolver, syntax defaults, tokens
├── main/
│   ├── index.ts                  # App lifecycle, window creation, hardening

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FaqFirebase/pi-desktop](https://github.com/FaqFirebase/pi-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
