---
trigger: always_on
description: Context for working on **SpecLens** with Claude Code. Read this before making changes.
---

# CLAUDE.md

Context for working on **SpecLens** with Claude Code. Read this before making changes.

## What it is

Desktop reader (Tauri 2 + React 19) for OpenSpec - the markdown convention with `proposal.md` / `tasks.md` / `specs/<capability>/spec.md` inside `<repo>/openspec/changes/<change-slug>/`. Archived changes live under `changes/archive/`.

Reads OpenSpec projects from local folders at runtime via a Tauri command. Users add folders themselves via "Add repository" in the sidebar; the list is persisted. The app starts empty and users point it at whatever they want. **GitHub integration is explicitly off the roadmap** - see the no-GitHub direction memory.

## Stack decisions

- **MUI + Emotion.** Confirmed UI stack. **Don't add Tailwind.**
- **Zustand + SQLite write-through (not `persist` middleware).** `useAppStore` holds UI state (theme, sidebar collapse, selected repo/change/tab, scroll target, `markdownZoom`, `highlightEars`, session-only panel state) **plus `repoSources: { path, missing }[]`** (the user-added folder list - paths persist, `missing` resets to `false` on cold start) **plus `settings: AppSettings`**. Persistence is done manually in `src/store/bootstrap.ts`: `bootstrap()` hydrates from SQLite on start, then `attachWriteThrough()` subscribes to each persisted slice and writes it back (UI keys → `kv_state`, sources → `repo_sources`). The loaded `repos: Repo[]` is **not** persisted - `reloadAllSources()` re-walks each path on mount. `useCommentsStore` persists via SQLite (`comments` table). `useAiStore` holds AI generation state + session summary caches.
- **Settings.** `AppSettings` + `DEFAULT_SETTINGS` + `HIGHLIGHT_COLORS` + `sanitizeSettings()` live in `useAppStore.ts`; the whole object persists as one `"settings"` kv blob. Mutate via `setSetting(key, value)` / `resetSettings()`. The dialog is in `src/sidebar/SidebarFooter.tsx` (General / Reading / AI tabs). **To add a setting:** extend the type + defaults, add a validated branch in `sanitizeSettings`, read `s.settings.<key>` at the consumer, add a control to the dialog - no new subscription needed.
- **Tauri `load_repo(path)` command** in `src-tauri/src/lib.rs` loads one project: walks its `openspec/` subtree, reads markdown + yaml, and (when a `.git/` exists at the project root or its immediate parent - see `find_git_root`) derives `DocAuthorship` per file via the `git2` crate (vendored libgit2 - no git binary needed). Authorship comes from **one history walk for all files** (`collect_file_histories`) - see "Authorship pipeline". The git-root walk is intentionally capped at one level up so that loading a folder from inside an unrelated git checkout (e.g. somewhere under `~`) doesn't pull authorship from that repo. JS calls the command once per source and catches per-source errors to mark `missing: true`. **Git is optional** - when absent, `Change.authorship`, `createdAt`, and `archivedAt` are `null` and the UI degrades gracefully.
- **Per-repo cold-start cache.** Each `load_repo` response includes a `signature` (git: `HEAD-sha + scoped porcelain status` hash; non-git: hash of `openspec/` file mtimes). The fast Tauri command `repo_signature(path)` returns the signature alone (no file reads). On cold start, `reloadAllSources` fetches the signature first; if it matches the cached entry in SQLite (`repo_cache` table, keyed by path), the saved `Repo` is used as-is - no walking, no git log. Mismatch → full reload + cache overwrite. See `src/lib/repoCache.ts` (thin wrapper over `src/lib/db.ts`). Dates in cached entries get revived (JSON round-trip loses the `Date` type). `useRepoSyncWatcher` polls the selected repo's signature (15s + window focus) and only *marks* it stale - reload is always user-initiated.
- **Local AI (`src-tauri/src/ai.rs`).** On-device summaries via llama.cpp (Metal) or an Ollama backend, exposed as `ai_*` commands. Nothing downloads or runs until the user fetches a model, so the no-network promise holds. Prompts are built on the JS side (`src/lib/aiSummary.ts`, `aiDocSummary.ts` - pure, unit-tested); Rust just receives the final string.
- **Spec checks (deterministic lint, labelled Beta).** `src/lib/specChecks.ts` is the engine (structural SL00x errors, consistency SL01x warnings, language SL02x checks) covering active change deltas + canonical `specs/<cap>/spec.md`; `src/lib/specChecksConfig.ts` is the registry - ids, severities, titles, message templates, and word lists all live there, the engine holds only detection logic. Both modules must stay pure (no Tauri imports) so the lint core can be extracted into a CLI later (see `docs/design/checks-and-claims.md`). Surfaced as: a Checks navigation view (two-level tree, by change or by check), a drag-resizable right panel that scopes to the open change/spec and hides on non-checkable views, in-document wavy underlines with hover diagnostics, list badges, and a ChangeViewer banner for findings with no text anchor. **All UI reads results through `src/specs/useSpecChecks.ts` (`useSpecCheckResults`)** - never call `runSpecChecks` from a component directly, the hook owns the settings wiring (`specChecks`, default on; `specChecksIncludeArchived`, default off).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dansreis/speclens](https://github.com/dansreis/speclens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
