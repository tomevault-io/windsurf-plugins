---
trigger: always_on
description: herdr plugin (Rust) that names a herdr pane from the coding agent's
---

# CLAUDE.md

herdr plugin (Rust) that names a herdr pane from the coding agent's
first prompt. When the pane is in an auto-generated linked worktree, it also
renames the worktree branch and workspace. The naming engine is swappable. The
default chain uses headless Pi first and then free OpenCode models; Apple
FoundationModels, Codex, and Claude remain explicit engine selections.

## Architecture

Single binary, two phases (`src/main.rs`):

- **Hot phase** (default, every `pane.agent_status_changed` event): pure env-var
  reads, no I/O. `context::evaluate` bails unless the new status is `working` and
  this pane does not already have a done marker. On a pass, writes a pane-scoped
  claim marker and forks the cold phase detached (`setsid`).
- **Cold phase** (`HERDR_NAMING_PHASE=cold`): `herdr::poll_agent_session` →
  `transcript::read_first_prompt` → `main::generate_name` (walks the
  `engine::engine_chain`; fallbacks `slug::display_fallback` for labels and
  `slug::fallback_from_prompt` for branch slugs) → renames configured targets.
  The generated name is also reported as the `task` metadata token on the pane
  and workspace for custom Agent and Space sidebar rows.
  With the default `tab` target on a non-linked workspace, the **title is
  written to `workspace`** (herdr 0.7.4's only bright Agent-sidebar token) and
  the **tab gets the cwd folder basename** (dim). Linked worktrees skip that
  title-on-workspace step so the later branch-slug rename can own the label.
  Multi-tab siblings share one workspace label (last writer wins).
  If the pane is in a linked worktree whose current branch starts with
  `worktree/`, `git::rename_current_branch` renames it to `<prefix>/<slug>` and
  only then `herdr::workspace_rename` renames the workspace to `<slug>`.

Naming outputs: title on workspace (non-worktree); folder on tab; optional
pane/agent renames via `targets`; `$task` metadata always. Branch
`<prefix>/<slug>` (bare `<slug>` when no prefix); workspace `<slug>` after a
successful worktree branch rename. The prefix comes from
`main::resolve_branch_prefix`: `HERDR_NAMING_BRANCH_PREFIX` env, then a
`branch-prefix` file in `HERDR_PLUGIN_CONFIG_DIR`, else none.

`zh` style: CLI engines return a Chinese label + ASCII slug; local fallback
prefers a compact CJK topic (strips spoken fillers) over ASCII kebab when the
prompt has hanzi.

Foundation-generated slugs should be compact noun-topic labels, not literal
sentence summaries. Prefer labels such as `current-file` over
`change-selected-file-to-current`. The helper must ground labels in the actual
prompt and avoid introducing absent concepts from examples or instructions.
The Foundation path is two-pass: generate several candidates, sanitize and
dedupe them, then ask FoundationModels to select exactly one candidate from the
cleaned list. Failure advances to the next configured engine, or the local
deterministic fallback when no engine remains.

## Naming engines

`generate_name` (in `main.rs`) walks an ordered chain from `engine::engine_chain`,
selected by `HERDR_NAMING_ENGINE`, and uses the first model that returns a valid
name and slug:

- unset / unknown → `[Pi, Opencode]`
- `pi` uses its configured default or walks a configured model list; `opencode`
  walks its free-model fallback list
- `foundation`, `codex`, or `claude` can still be selected explicitly

Each model attempt yields `None` on command failure, timeout, empty output, or
invalid naming output. The chain degrades cleanly to the deterministic local
slug. Pi and OpenCode attempts have a 15-second per-model ceiling.

**OS gate:** the `Foundation` engine is `#[cfg(target_os = "macos")]`. Off macOS
(e.g. Linux) the enum variant, the `foundation` module, and the matching
`[[build]]` swift step are all compiled/skipped. The default remains
`[Pi, Opencode]`; a `foundation`-only request resolves to that default chain
when Foundation is unavailable. The plugin's
`platforms` are `["macos", "linux"]` (Unix only; the cold phase detaches via
`setsid`). Verify the Linux build with
`cargo check --target x86_64-unknown-linux-gnu`.

## Module map

- `context.rs` — parse the two env JSON blobs, working-status eligibility gate
- `slug.rs` — `sanitize` + `fallback_from_prompt`
- `engine.rs` — pure `engine_chain(HERDR_NAMING_ENGINE)` → ordered fallback list
  (OS-aware: Foundation only on macOS)
- `transcript.rs` — resolve transcript path (glob) + first-prompt extraction for
  `claude` and `codex` (different on-disk formats). Claude slash-command
  wrappers are used as a fallback naming prompt, including `command-args`, when
  no normal non-meta user prompt exists; expanded skill bodies remain ignored.
- `foundation.rs` — macOS-only (`#[cfg(target_os = "macos")]`) on-device engine;
  builds a bounded head/tail prompt excerpt, shells to the `herdr-namer` Swift
  helper (15s timeout), sanitizes its stdout
- `codex.rs` — `codex exec --ignore-user-config --ephemeral -s read-only` with a 30s timeout
- `herdr.rs`: `herdr pane get` (polled), pane/workspace task metadata, and
  pane/workspace renames
- `git.rs` — current branch + `git branch -m`
- `naming-helper/` — SwiftPM package (`herdr-namer`): two FoundationModels
  guided-generation calls. The first fills a `@Generable TaskNameCandidates`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wenhanweime/herdr-plugin-renamer](https://github.com/wenhanweime/herdr-plugin-renamer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
