---
trigger: always_on
description: Terminal code-review companion for AI agents. Launched in a repo, it renders a
---

# diffler: agent guide

Terminal code-review companion for AI agents. Launched in a repo, it renders a
live neogit-style git UI and embeds an MCP server, so an agent reads review
comments in place, replies, and reacts to feedback. The human reviews and drives
git; the agent responds; the diff updates live. Philosophy: YAGNI/KISS (one
small native binary, alternate-screen TUI, no daemon, no browser).

## Layout

```
crates/diffler-core/   pure logic, no terminal (errors via thiserror):
  vcs.rs / git.rs      Vcs trait + git2 backend (status, diff, log, stage, commit, branch)
  jj.rs                JjVcs: composes the git2 backend for reads, shells out to `jj` for writes
  repo.rs              repository discovery and backend selection (git vs colocated jj)
  model.rs diff.rs     diff model, hunks
  diffalgo.rs          selectable line-diff algorithms (myers/minimal/patience/histogram/structural)
  pairing.rs           similarity line-pairing + grapheme intraline emphasis
  syntax/              tree-sitter language registry + AST-diff intraline emphasis + scope index
  highlight.rs         syntect whole-file highlight
  source.rs review.rs  ReviewSource + per-source review state
  session.rs           comments (a walkthrough stop is one) + viewed marks
  walkthrough.rs       the agent's reading order: the stops' comment ids, anchors
  store.rs             .diffler/ persistence
  feedback.rs          markdown feedback export

crates/diffler/        binary (color-eyre at the top; thiserror for typed errors):
  ui/ app/ tree.rs     ratatui TUI: screens, file sidebar, state
  app/composer.rs      in-place comment editor (app/text_edit.rs is its key set)
  ci/                  forge seam: CI acquisition + PR review (ForgeProvider trait; gh/glab/Forgejo REST)
  graph/               navigable orthogonal node-graph ratatui component
                       (mermaid.rs parses the flowchart subset an agent writes;
                       sequence.rs and callstack.rs are the other two figure
                       kinds a walkthrough card can draw, drawing.rs picks
                       between all three by a fence's own language and header)
  keymap.rs config.rs  configurable keybindings, layered TOML config
  theme.rs transient.rs  rendering theme, popup/modal model
  mcp.rs               rmcp/axum MCP server
  watch.rs             notify filesystem watcher
  editor.rs clipboard.rs  $EDITOR suspend/restore, OSC52 yank
  text.rs               display-width text shaping shared by the UI and the graph engine
```

## Commands (just; see `just --list`)

- `just check`: clippy with ci's denials, run after every change
- `just test`: nextest + doctests (needs `jj` on PATH for the jj backend's integration tests)
- `just fix`: clippy --fix + fmt
- `just snap`: insta snapshot tests; read `.snap.new` diffs before `just snap-accept`
- `just e2e`: PTY end-to-end suite (needs `uv` and `jj`; CI runs it in a separate job)
- `just package-check`: what crates.io builds. A crate packages only its own
  directory, so a file it reaches outside one builds here and fails the publish
  after the tag is public. `just ci` carries the include rule; the release
  script runs the whole check.
- `just ci`: fmt+clippy+tests gate, must pass before any commit (CI additionally runs msrv, deny, typos, dupes, machete, coverage)
- `showcase/record.sh`: regenerate `showcase/img/*.png`, one screenshot per theme
  (needs `vhs`). It seeds a throwaway repo with a three-file review and shoots
  the review screen with all three panes up, the comments sidebar included, so
  rerun it after anything that changes how that screen looks. `record.sh --seed`
  prints the seeded repo and records nothing, for checking the frame first.
  The README's hero `assets/demo.gif` is hand-recorded and has no script.

## Rules

- Code is done only when `just ci` passes. Run it, don't assume.
- No `unwrap`/`panic!`/`todo!` in non-test code (clippy denies). `expect` needs justification.
- Clippy scores every function's cognitive complexity and warns over 20, which
  is a warning CI denies. The two dispatch loops above it carry an allow and a
  reason; a third means the function grew a second job, so split it.
- Errors: `thiserror` for typed library-style errors (diffler-core, the `ci` module); `color-eyre` for the binary's top level only.
- No `println!`/stdout writes in the TUI (corrupts the screen; clippy denies it).
- Async: never block in async fns; `spawn_blocking` for CPU/IO-heavy work.
- TUI changes need TestBackend + insta snapshot coverage. A changed snapshot is a
  behavior change: read the diff, never accept blindly, never edit `.snap` by hand.
- Run `just e2e` after rendering/behavior changes: `just ci` skips it, and glyph
  or timing changes can pass ci yet break the PTY suite.
- PTY e2e probes must drain output continuously (the suite's wait helpers do);
  a bare sleep fills the PTY buffer and freezes the app under test.
- Test fixtures and sample data use generic mock names ("reviewer",
  "acme/widgets"), never real usernames, handles, or emails.
- Hooks are managed by prek (`prek install` once). If a hook fails, fix the cause.
  Never `git commit --no-verify`.
- Review before committing: in Claude Code run `/rev` on the working tree for any

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matheusfillipe/diffler](https://github.com/matheusfillipe/diffler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
