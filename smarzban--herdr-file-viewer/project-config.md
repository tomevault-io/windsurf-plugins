---
trigger: always_on
description: Stranger litmus test: would this instruction make sense to a stranger who cloned this repo? If
---

# AGENTS.md

## Routing guideline

Stranger litmus test: would this instruction make sense to a stranger who cloned this repo? If
no, it belongs in AGENTS.local.md.

A gitignored AGENTS.local.md may exist beside this file; if present, read and follow it before starting work.

Pointer files carry no content: edits go to AGENTS.md or AGENTS.local.md, never CLAUDE.md: it is a
frozen one-line pointer and says so in-file.

Lazy creation: if an agent has private-routed content (per the litmus test above) and no
AGENTS.local.md exists yet in this working copy, it creates one; the committed .gitignore entry
already covers it, so the pattern self-propagates to every clone.

@AGENTS.local.md

## Project overview

**Cross-agent source of truth for this repo.** Any coding agent (Claude Code, Cursor, Codex,
Aider, …) should read this first. It is intentionally vendor-neutral: agent-specific entry files
(e.g. `CLAUDE.md`) import or point at this file rather than duplicating it.

> **Maintainability rule:** standing project rules live HERE, once. Don't copy them into per-agent
> files. Those should be thin shims that `@import`/reference this.

Companion docs:

- **`CONTEXT.md`**: the glossary (canonical vocabulary).
- **`constitution.md`**: the standing principles (the source for "Load-bearing constraints").
- **`ARCHITECTURE.md`**: the committed module map (keep it current when components change).
- **`docs/`** (index: `docs/README.md`): the user-facing docs — `keys.md` (the full key/mouse
  reference), `configuration.md` (the `config.toml` + `[keys]` reference), `usage.md` (per-feature
  guide), plus `install.md` / `summoning.md` / `renderers.md` / `windows.md`. The root `README.md` is
  a lean front door that links into these; reference detail lives in `docs/`, never the README.

### What this is

A **herdr plugin**: a git-aware, read-only **file viewer**: a keyboard-driven TUI that opens in a
herdr split pane, with a directory tree on the left and a content pane on the right (rendered
markdown, diffs, or syntax-highlighted content). herdr is the host (a Rust+ratatui terminal agent
multiplexer); this plugin is built to align with it.

### Current state: BUILT & SHIPPED

The plugin is fully built and shipped publicly to **`smarzban/herdr-file-viewer`**. `Cargo.toml`,
`src/` (lib + modules + thin binary), `herdr-plugin.toml`, CI, and tagged releases all exist.
`main` is **protected** (PR + green CI required; force-push/delete blocked).

### Architecture (the big picture)

A **single in-process TUI owns both columns** (ADR-0002). It is not composed of multiple herdr
panes. Logical components and their one-line responsibilities (full contracts in `ARCHITECTURE.md`
and the spec chain):

- **Host Adapter**: the herdr boundary: manifest declaration + parsing injected context + open-pane requests
- **Root Resolver**: resolve the tree root (worktree root vs cwd) and git-presence
- **Tree Model**: the rooted, gitignore-aware file tree + filters + cursor
- **Git Service**: read-only git queries (status, baseline, changed-set, diff)
- **View Policy**: pure decision: which view mode for a file (changed→diff, md→rendered, else→content)
- **Official Repository Gateway**: fixed-source, bounded, display-only remote notices
- **Content Renderer**: produce content-pane text by delegating to external CLIs, with guards
- **Presenter**: draw the two-column layout (ratatui)
- **Input Dispatcher**: map key events → intents (crossterm)
- **Session Controller**: orchestrate intents → state changes; holds in-memory session state
- **Editor Launcher**: hand a file off to an external editor in-process, suspending and resuming the
  TUI around it (NOT a herdr pane — see the herdr integration section)

State is **in-memory and ephemeral only** except for the safe-to-delete, advisory
`update-check.json` cache, which never changes the viewed root or git repo.
(`ARCHITECTURE.md` is the committed module map; keep it current.)

### Load-bearing constraints (from `constitution.md`)

These shape every decision; violating one is a design error, not a style nit:

- **Read-only.** No file or git mutations. The editor path is hand-off only. (AC-N1, AC-N2)
- **Delegate rendering.** Reuse external CLIs (`glow` markdown, `delta` diff, `bat` syntax); build
  only the shell. Never reinvent rendering. (ADR-0001)
- **Git is first-class**, woven through the tree and content pane, not a separate mode.
- **Keyboard-first.** Every function reachable by keyboard; no mouse required. (AC-18)
- **Good plugin citizen.** Drive herdr only through its documented CLI/socket; no persistent state
  beyond the plugin's own dirs.
- **YAGNI.** Smallest thing that meets the criteria; resist turning a viewer into a file manager or
  git client.

### Stack specifics

- **Rust 1.96 (edition 2024)** + **ratatui 0.30.x** (uses `ratatui-core` 0.1.x) + **crossterm 0.29.0**
- **`ansi-to-tui` 8.0.1** ingests the external renderers' ANSI output into ratatui spans, and
  doubles as the **AC-27 escape-neutralizer** (maps styling, drops cursor/screen-control). All file
  content flows through it.
- **`ignore` 0.4.26** for fast, `.gitignore`-aware tree walking (do not hand-roll gitignore).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [smarzban/herdr-file-viewer](https://github.com/smarzban/herdr-file-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
