---
trigger: always_on
description: > 📚 [INDEX](INDEX.md) · [Sync Checklist](AGENTS.md) · [Commands](commands.md) · [Server API](server_api.md) · [Stream Events](stream-events.md) · [str_func](str_func.md)
---

> 📚 [INDEX](INDEX.md) · [Sync Checklist](AGENTS.md) · [Commands](commands.md) · [Server API](server_api.md) · [Stream Events](stream-events.md) · [str_func](str_func.md)

# structure/ — Sync Guide

- The canonical desktop release workflow must Developer ID-sign macOS with Team `U9ATA49N28`, notarize, staple, verify the final app, and verify channel metadata plus the update ZIP SHA-512 before upload. `electron:dist:mac:signed` is the equivalent opt-in local path; ordinary `electron:dist:mac` remains ad-hoc. Windows remains unsigned. Keep README and `infra.md` aligned; fixture tests do not prove a signed/notarized artifact, and the first signed release is a manual-DMG bootstrap before in-app updates can be trusted.

- Auto (`permissions:auto`) grants qualified direct-local Jaw API authority across supported runtimes, independently of per-turn secrets. Keep actual/effective loopback, exact browser origin, proxy provenance, explicit outbound destinations and server-only resource options. Safe/custom keep existing scoped/operator paths; full API authority is instance-wide, distinct from provider Safe and task scope. Preserve no-descendant/read-only assignments, captured worker context and honest capability/receipt evidence. See `../docs/slack-tools.md` and `server_api.md`.
- Slack group DMs use `message.mpim`, which requires `mpim:history` — without it group DMs do not arrive at all; existing IM/channel installs keep working and merely report the optional capability gap. Exact `channel_type: mpim` mentions retain channel allowlist and thread policy, never the one-to-one DM bypass. an absent scope header is unknown and a present empty header is a known empty grant. Keep `telegram.md` and the validation API docs synchronized.

- Native Code: synchronize `runtime-integration.md`, `server_api.md`, `INDEX.md` and root guides for `src/code-mode/` and `/api/code`. Code uses separate per-backend storage and direct native adapters; preserve append/status replay, complete active snapshots, byte budgets, early resource registration and physical-exit proof. Interruption must seal callbacks before owner-checked accepted-buffer persistence; both hosts use `src/routes/code-body-parser.ts` for Code envelope/decoded limits. Compact replay has one carve-out: a Claude rollback deletes the removed items and their events, so replay from below its `replay_floor_sequence` answers `invalid_sequence` and a higher `historyGeneration` means a new snapshot. Private prompt boundaries never reach the wire, the fork is verified before one CAS commit, and the source native session is never changed.

- Linux `/api/file/open` acknowledges asynchronous `xdg-open` launch, not desktop application success. Keep detached/ignored-stdio dispatch and launch-error handling; never wait synchronously for the opener. See `server_api.md`.

- Sidecar build/smoke: sync `infra.md` and root notes for transactional source/stage/lock ownership, runtime-candidate/seal matching, outside-checkout target-Node execution, preserved asset/prune/native/no-JWC gates and explicit retained evidence/cleanup. Ordinary import, live server readiness and final packaged native UI are separate proofs; no timeout or skipped green.

- Isolated desktop QA: `src/shared/isolated-qa.ts` owns opt-in role paths/strict ports/child env; Electron, dashboard CLI and Manager enforce the captured launch policy before their owned side effects. Keep the supervisor-before-import boundary, no global registration/installer or foreign scan/peer/lifecycle actions, normal-mode compatibility and lifetime-safe QA cleanup explicit in `infra.md` and root docs. Do not conflate mocked/compiled launch checks with final packaged native UI proof.

- Keep this folder aligned with the live `cli-jaw` tree; `INDEX.md` lists the public architecture docs and support tools.
- Private plans, audits, evidence, and history belong only in a separate sibling clone of [cli-jaw-internal](https://github.com/lidge-jun/cli-jaw-internal); request access through an [issue](https://github.com/lidge-ai/cli-jaw/issues). Never create private records inside this checkout, including `devlog`, `_plan`, `_fin`, or `.jwc` aliases at any depth, even when generic skill defaults suggest them. `docs/` and `structure/` are for public product documentation; omit private record paths from public docs and source.
- Update `INDEX.md` whenever a doc is added, removed, renamed, or re-scoped. Keep the doc map, tier list, and quick links in sync.
- Update `str_func.md` file-tree entries when files are added, removed or renamed in `server.ts`, `src/routes/*`, `src/cli/handlers*.ts`, `src/cli/api-auth.ts`, `src/manager/*` (multi-instance dashboard), `bin/commands/*`, `bin/star-prompt.ts`, `tests/`, `public/`, or generated-dist exclusions change. `verify-counts.sh` checks that every file-tree entry in `str_func.md` points at a real file (it records no line counts).
- `stream-events.md` is the SSE/WS/event-trace companion for `frontend.md`, `server_api.md`, and the ProcessBlock pipeline. Keep `GET /api/events`, replay behavior, and fallback WS current.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lidge-ai/cli-jaw](https://github.com/lidge-ai/cli-jaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
