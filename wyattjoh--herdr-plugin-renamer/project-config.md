---
trigger: always_on
description: herdr plugin (Rust) that names a herdr pane from the coding agent's
---

# CLAUDE.md

herdr plugin (Rust) that names a herdr pane from the coding agent's
first prompt. When the pane is in an auto-generated linked worktree, it also
renames the worktree branch and workspace. The naming engine is swappable:
on-device Apple FoundationModels by default (a small Swift helper), with a
headless Codex call as the automatic fallback.

## Architecture

Single binary, two phases (`src/main.rs`):

- **Hot phase** (default, every `pane.agent_status_changed` event): pure env-var
  reads, no I/O. `context::evaluate` bails unless the new status is `working` and
  this pane does not already have a done marker. On a pass, writes a pane-scoped
  claim marker and forks the cold phase detached (`setsid`).
- **Cold phase** (`HERDR_NAMING_PHASE=cold`): `herdr::poll_agent_session` →
  `transcript::read_first_prompt` → `main::generate_slug` (walks the
  `engine::engine_chain`; fallback `slug::fallback_from_prompt`) →
  `herdr::pane_rename`. The generated slug is also reported as the `task`
  metadata token on the pane and workspace for custom Agent and Space sidebar
  rows. If the pane is in a
  linked worktree whose current branch starts with `worktree/`,
  `git::rename_current_branch` renames it to `<prefix>/<slug>` and only then
  `herdr::workspace_rename` renames the workspace to `<slug>`.

Naming outputs: pane `<slug>`; branch `<prefix>/<slug>` (bare `<slug>` when no prefix is configured;
`main::compose_branch` joins them); workspace `<slug>` after a successful
worktree branch rename. The prefix comes from `main::resolve_branch_prefix`:
`HERDR_NAMING_BRANCH_PREFIX` env, then a `branch-prefix` file in
`HERDR_PLUGIN_CONFIG_DIR`, else none.

Foundation-generated slugs should be compact noun-topic labels, not literal
sentence summaries. Prefer labels such as `current-file` over
`change-selected-file-to-current`. The helper must ground labels in the actual
prompt and avoid introducing absent concepts from examples or instructions.
The default Foundation path is two-pass: generate several candidates, sanitize
and dedupe them, then ask FoundationModels to select exactly one candidate from
the cleaned list. Codex remains a fallback only when Foundation fails.

## Naming engines

`generate_slug` (in `main.rs`) walks an ordered chain from `engine::engine_chain`,
selected by `HERDR_NAMING_ENGINE`, and uses the first engine that returns a slug:

- unset / `foundation` / unknown → `[Foundation, Codex]` (on-device first)
- `codex` → `[Codex]` only

Each engine returns `Option<String>` and yields `None` on any failure, so the
chain degrades cleanly: Foundation → Codex → deterministic local slug. Engine
binaries are overridable via `HERDR_NAMING_FOUNDATION_BIN` and
`HERDR_NAMING_CODEX_BIN`.

**Platform/capability gate:** the `Foundation` engine is
`#[cfg(target_os = "macos")]`. Off macOS (e.g. Linux) the enum variant, the
`foundation` module, and the matching `[[build]]` step are skipped, so the
default chain collapses to `[Codex]` and a `foundation` request is silently
downgraded. On macOS, `naming-helper/build-if-supported.sh` builds the helper
only when the active Swift toolchain supports `@Generable` / `@Guide`; a helper
build failure after a successful probe remains fatal. The plugin's platforms
are `["macos", "linux"]` (Unix only; the cold phase detaches via `setsid`).
Verify the Linux build with `cargo check --target x86_64-unknown-linux-gnu`.

## Module map

- `context.rs` — parse the two env JSON blobs, working-status eligibility gate
- `slug.rs` — `sanitize` + `fallback_from_prompt`
- `engine.rs` — pure `engine_chain(HERDR_NAMING_ENGINE)` → ordered fallback list
  (OS-aware: Foundation only on macOS)
- `transcript.rs` — resolve transcript path (glob for Claude/Codex; reported
  session path for Pi) + first-prompt extraction. Claude slash-command wrappers
  are used as a fallback naming prompt, including `command-args`, when no normal
  non-meta user prompt exists; expanded skill bodies remain ignored.
- `foundation.rs` — macOS-only (`#[cfg(target_os = "macos")]`) on-device engine;
  builds a bounded head/tail prompt excerpt, shells to the `herdr-namer` Swift
  helper (15s timeout), sanitizes its stdout
- `codex.rs` — `codex exec --ignore-user-config --ephemeral -s read-only` with a 30s timeout
- `herdr.rs`: `herdr pane get` (polled), pane/workspace task metadata, and
  pane/workspace renames
- `git.rs` — current branch + `git branch -m`
- `naming-helper/` — SwiftPM package (`herdr-namer`): two FoundationModels
  guided-generation calls. The first fills a `@Generable TaskNameCandidates`
  with candidate slugs; the helper sanitizes and dedupes them. The second fills
  a `@Generable SelectedTaskName` by copying one cleaned candidate. The helper
  prints the selected bare slug on stdout (exit 0), or a reason on stderr
  (non-zero) when Apple Intelligence is unavailable or generation fails. Same
  stdout-or-fail contract as `codex`.

## Conventions

- Fail open: every path exits 0; never block herdr.
- First-prompt idempotence uses a short-lived pane claim to block duplicate cold
  phases and a session-scoped done marker to block later events for the same
  native agent session.
- The cold phase polls for BOTH the session and the first prompt. Claude reports

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wyattjoh/herdr-plugin-renamer](https://github.com/wyattjoh/herdr-plugin-renamer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
