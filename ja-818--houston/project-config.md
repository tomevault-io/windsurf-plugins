---
trigger: always_on
description: Caveman style. Progressive discovery. Load on demand.
---

# Claude Session Protocol — Houston

Caveman style. Progressive discovery. Load on demand.

---

## RULE 0 — NEVER TAKE SHORTCUTS

**This is the highest rule. It outranks every phase, every dispatch, every "be efficient" instinct.**

If a faster path and a better path both exist, the better path wins. Always. Even if "better" means 40 directory renames, 28 cross-reference fixes, a doc update, a test rewrite, and a hash bump. Volume of work is not a reason. "It would still work" is not a reason.

**Watch for these shortcut patterns in your own thinking:**

- "This avoids the rename / refactor / doc update / migration." → That's labor, not a reason. Do the work.
- Adding an override / opt-out / fallback field to paper over a bad name or bad shape. → Fix the underlying name or shape.
- Skipping a typecheck / test / build because it's slow or set-up-heavy. → Run it.
- Keeping legacy compatibility "just in case" inside internal code. → Delete it (per the existing no-backwards-compat rule).
- Generating bulk content with a script when each item deserves real thought. → Do the items by hand.
- Writing a TODO / FIXME / "we can clean this up later." → Clean it up now.
- Declaring something done before the verification step actually passed. → Not done.
- Picking the smallest example to demo on without asking the user. → Ask.

**Self-audit at the end of every meaningful chunk of work:**
> "Where did I cut a corner? Was it because the better path was wrong, or because it was just longer?"
>
> If "just longer" — go back and do it right.

This rule was added because the model defaulted to a shortcut (a `display_name` schema override instead of renaming 40 slugs) and the user had to push back. Don't make the user push back. Catch yourself first.

---

## PHASE 0 — Load /caveman (EVERY SESSION, FIRST)

Before any action: invoke `/caveman` skill. Stay terse. Drop articles, filler, pleasantries. Technical substance stays. Code blocks unchanged.

Default level: `full`. Switch via `/caveman lite|full|ultra`.

Off only if user says "stop caveman" or "normal mode".

---

## System at a glance (read once at session start)

Houston = desktop app + standalone engine + open library of agents.

- **`app/`** — Tauri 2 desktop. React frontend, small Rust binary that spawns the engine as a sidecar subprocess and talks to it over HTTP/WS. OS-native glue only (file pickers, reveal-in-finder, logs). No domain logic.
- **`engine/`** — Rust crates. `houston-engine-core` = runtime/domain. `houston-engine-protocol` = wire types. `houston-engine-server` = axum HTTP+WS binary (`houston-engine`). `houston-agent-files`, `houston-skills`, `houston-sessions`, `houston-file-watcher`, etc. are leaf crates. Frontend-agnostic: no Tauri, no React.
- **`ui/`** — `@houston-ai/*` React packages (chat, board, layout, engine-client, …). Props-only, no store imports. `@houston-ai/engine-client` is the TS front door to the engine.
- **User data** — `~/.houston/`: DB, logs, `engine.json`, and `workspaces/<Workspace>/<Agent>/`. Each agent has `.houston/` data files + `CLAUDE.md` + `.agents/skills/`.
- **Wire contract** — every domain call is a `fetch` or WS frame in `@houston-ai/engine-client`. There are NO `invoke("list_workspaces", …)` style Tauri commands for domain; those were all deleted.
- **Reactivity** — engine emits `HoustonEvent`s; desktop subscribes to the WS `*` firehose; TanStack Query invalidation in `app/src/hooks/use-agent-invalidation.ts` maps events → query keys. File watcher catches direct agent writes.
- **Voice** — agents' target user is NON-technical. The product system prompt forbids mentioning files/JSON/configs/CLIs when talking to the user. Lives in `app/src-tauri/src/houston_prompt.rs` (Houston app), NOT in the engine. Engine is prompt-agnostic; app hands it over at spawn via `HOUSTON_APP_SYSTEM_PROMPT`.

Before touching anything: run PHASE 1 (load `knowledge-base/architecture.md` + any KBs relevant to scope).

## Dispatch table (progressive discovery)

Deploying / shipping a release? → `/release`
Manual macOS build, notarize, staple? → `/build-app-local`
Bug? Don't guess → `/debug`

Need specific knowledge? Load on demand:
- Repo shape, products, engine story → `knowledge-base/architecture.md`
- Colors, typography, components, animation → `knowledge-base/design-system.md`
- `.houston/` layout, schemas, reactivity → `knowledge-base/files-first.md`
- Skills on disk / "Actions" in UI, picker, invocation marker → `knowledge-base/actions.md`
- Agent manifest, tiers, sidebar, workspaces → `knowledge-base/agent-manifest.md`
- Engine wire protocol (REST + WS) → `knowledge-base/engine-protocol.md`
- `houston-engine` binary ops → `knowledge-base/engine-server.md`
- Bundled CLIs (codex universal, composio per-arch) + runtime claude-code installer → `knowledge-base/cli-bundling.md`
- Custom frontend on `houston-engine` (integration reference) → `examples/smartbooks/README.md`
- Mobile PWA (tunnel, pairing, reactivity) → `docs/mobile-architecture.md` + `docs/relay-operations.md`
- Updater, analytics, Sentry, env vars, CI → `knowledge-base/production-infra.md`
- Supabase auth, Google SSO, Keychain → `knowledge-base/auth.md`
- Translating UI strings, namespaces, ui/ labels prop pattern, `t()` rules → `knowledge-base/i18n.md`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ja-818/houston](https://github.com/ja-818/houston) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
