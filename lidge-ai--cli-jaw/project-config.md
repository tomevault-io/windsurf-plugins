---
trigger: always_on
description: - The canonical desktop release workflow must Developer ID-sign macOS with Team `U9ATA49N28`, notarize, staple, verify the final app, and verify channel metadata plus the update ZIP SHA-512 before upload. `electron:dist:mac:signed` is the equivalent opt-in local path; ordinary `electron:dist:mac` remains ad-hoc. Windows remains unsigned. Keep README and `structure/infra.md` aligned; fixture tests do not prove a signed/notarized artifact, and the first signed release is a manual-DMG bootstrap bef
---

# CLI-JAW Claude Guide

- The canonical desktop release workflow must Developer ID-sign macOS with Team `U9ATA49N28`, notarize, staple, verify the final app, and verify channel metadata plus the update ZIP SHA-512 before upload. `electron:dist:mac:signed` is the equivalent opt-in local path; ordinary `electron:dist:mac` remains ad-hoc. Windows remains unsigned. Keep README and `structure/infra.md` aligned; fixture tests do not prove a signed/notarized artifact, and the first signed release is a manual-DMG bootstrap before in-app updates can be trusted.

This repository is a Node.js ESM orchestration runtime for boss/employee dispatch, Web UI, browser/CDP automation, Telegram/Discord/Slack channels, memory, heartbeat, and PABCD orchestration.

The `/api/code` API owns isolated Codex/Claude/Cursor/Grok sessions through `src/code-mode/host.ts`. Use its dedicated store, native adapters and captured turn/resource ownership; keep full snapshots, compact replay and byte limits synchronized with the runtime and API architecture docs. Claude conversation rollback is the one replay carve-out: removed items and their events are deleted, replay below the replay floor answers `invalid_sequence`, and a higher `historyGeneration` requires a new snapshot.

Native Code interruption seals callbacks before persisting accepted buffered content under the captured store owner. Worker and Manager share the Code JSON body policy in `src/routes/code-body-parser.ts` (1MiB decoded prompt, 6MiB + 4KiB envelope); generic API limits stay separate. Settings navigation guards the drafts actually being discarded, including keyboard and desktop subscriptions.

## Documentation Map

- Start at `structure/INDEX.md` for the current architecture map.
- Architecture contract notes live in `structure/AGENTS.md` §Root contract notes.
- Workbench modernization uses a one-row Activity header with Codex-style expanded rows/groups; the Workbench Settings tab is replaced by a ZCode-style full settings page that swaps the workspace (header gear / Meta+,, `← Back to workspace`, grouped icon nav, card content), persisted as Manager registry `ui.instanceSettingsOpen`. A unified settings registry separates Instance/Manager scopes and the same page is served standalone from `dist/settings` behind the Classic header gear (the right-panel 설정 tab is gone); Classic uses the t3 token shell. Preserve per-page save owners, dirty guards, Preview iframe identity and independent live Requests; see `structure/frontend.md`.
- Keep `README.md`, `AGENTS.md`, this file, and `structure/AGENTS.md` aligned when command/API/orchestration behavior changes. Concurrent inbound gateway changes belong in `structure/INDEX.md`, `structure/infra.md`, `structure/telegram.md`, and the messaging runtime docs.
- `docs/` and `structure/` contain public product documentation only. Private plans, audits, evidence, and history belong only in a separate sibling clone of [cli-jaw-internal](https://github.com/lidge-jun/cli-jaw-internal); request access through an [issue](https://github.com/lidge-ai/cli-jaw/issues).
- Never create private records inside this checkout, including `devlog`, `_plan`, `_fin`, or `.jwc` aliases at any depth. This overrides generic skill defaults. Do not include private record paths in public docs or source.
- Before public pushes, check the index with `npm run check:private-boundary` and outgoing commit trees with `node scripts/check-private-boundary.mjs --range <remote-base> HEAD`; enable the checkout-local pre-push hook as described in [CONTRIBUTING.md](CONTRIBUTING.md#local-private-path-check). Review content separately. CI runs after upload and cannot prevent initial disclosure.

- Manager sidebar selection and Sessions/Stop/Open use separate interactive targets. Keep list navigation scoped to the focused row selector, stable session-disclosure links, and preferred width separate from viewport clamping. Pointer and keyboard resize completion persist the latest value; see `structure/frontend.md`.
- Manager terminal presentation preserves backend PTY ownership across hide/unmount. Keep hydration-first bounded creation, explicit recovery, stable tab identity and focus ownership separate from native Code API sessions; theme updates must not recreate shells. See `structure/frontend.md`.

## Build & Deploy Contract

- The running server executes compiled `dist/` (`jaw serve` → `dist/server.js`), never the TS sources. After changing `server.ts`/`src/**`/`bin/**`, run `npm run build` before telling anyone to restart; frontend changes additionally need `npm run build:frontend`. Full rules: `AGENTS.md` § Build & Deploy Contract.

## Current Runtime Notes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lidge-ai/cli-jaw](https://github.com/lidge-ai/cli-jaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
