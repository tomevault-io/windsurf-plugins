---
trigger: always_on
description: `hera` (repo: hera-agent-godot) is a low-token CLI that lets you inspect and
---

# Working with hera-agent-godot (for AI agents)

`hera` (repo: hera-agent-godot) is a low-token CLI that lets you inspect and
control a **live Godot 4.x editor**. Use it to act on the *real* editor and
check the result — don't guess scene structure or whether a change worked from
memory. The binary installs as `hera`; `hera-agent-godot` is a transitional
alias for the same CLI.

## Co-developing this repo (Claude Code × Codex)

Claude Code and Codex collaborate on **developing hera-agent-godot itself**
(the Go CLI, the GDScript addon, docs, and distribution) — this section is
about building the tool, not driving the editor with it. Rules for both
agents:

- The other agent reconstructs context **only from the repo**: git history and
  docs. Make small commits with descriptive English messages; never leave
  meaningful state only in chat.
- State that lives **outside this repo** — Godot Asset Store submissions and
  the Homebrew tap repo ([NotNull92/homebrew-hera](https://github.com/NotNull92/homebrew-hera))
  — must be recorded under `docs/` (release notes in `docs/releases/` or the
  ROADMAP) whenever it changes.
- `main` is shared: never force-push or rewrite pushed history; rebase
  local-only work.
- The same gates apply regardless of agent: `go build/vet/test` + `gofmt`
  for Go; Godot `--check-only` for GDScript; sync README (EN/KO),
  `docs/COMMANDS.md`, and `docs/ROADMAP.md` when the surface changes, and
  regenerate the contract goldens when `docs/CONTRACT.md` behavior changes.
- Outward-facing actions (store uploads, PRs to third-party repos, version
  bumps/releases) need the user's explicit go-ahead; Asset Store form
  submission is done by the user personally.
- Machine-specific facts (Godot binary path, isolated-smoke pattern, toolchain
  limits, external-form gotchas) live in [docs/DEV_MACHINE.md](docs/DEV_MACHINE.md)
  — read it before local Godot smokes, third-party PRs, or publishes, and
  update it there instead of re-discovering.
- **Ported capabilities must be fully naturalized.** When an idea, architecture,
  or workflow is adapted from an outside tool, what ships is a Hera-native
  capability: named for the Godot/Hera construct it operates on, and justified
  from engine behaviour rather than from "the other tool does it this way".
  Leave no external tool name, no "ported from X" framing, no side-by-side
  comparison tables, and no borrowed taxonomy labels anywhere in the repo —
  docs, skills, and code alike. Re-derive each rule from the Godot fact that
  forces it; if a rule cannot be justified that way, it does not belong.
  Attribution cuts the other way for **copied material**: anything actually
  vendored or licensed keeps its upstream provenance and licence, and published
  standards stay cited (as WCAG is in the `ui-theme-qa` corpus). Never strip a
  credit while keeping copied expression — rewrite the expression genuinely
  instead. Prefer deriving values from Godot's own defaults over vendoring
  someone else's data, so the question does not arise.

## Canonical Godot sources for documentation and review

When verifying or reviewing Godot engine behavior, APIs, CLI flags, version
compatibility, or official documentation, consult the maintained upstream
repositories first:

- Godot engine: [github.com/godotengine/godot](https://github.com/godotengine/godot)
- Official Godot documentation: [github.com/godotengine/godot-docs](https://github.com/godotengine/godot-docs)

Local repository documentation remains authoritative for Hera-specific
contracts and policies. Use the upstream sources above to settle Godot facts;
do not substitute stale recollection or unofficial summaries when they apply.

## When to use it

- You need the actual state of the open scene (node tree, a node's properties).
- You want to change the scene (add/set/remove nodes) and confirm it stuck.
- You want to run a scene, then read the log for errors.
- You want an off-screen preview render of the edited scene (`screenshot`) or a
  live game viewport capture (`screenshot --runtime` / `game screenshot`).

If the user is not running the Godot editor with the **Hera Agent** plugin
enabled, commands fail with "no live Godot editor found" — ask them to enable it.

## Setup (once)

1. Open the project in a **Godot 4.x** editor.
2. Enable **Project → Project Settings → Plugins → Hera Agent Godot**. The Output
   panel should show `[hera] ... listening on 127.0.0.1:<port>`.
3. Get the CLI: install a release binary (see the README's Install section) or
   build from source with `go build -o hera .` (from the repo root).

The CLI finds the editor automatically via `~/.hera-agent-godot/instances/`.

## Commands

```
hera status                                  # project / version / active scene / UI mode
hera scene tree                              # node tree of the edited scene
hera scene list                              # open scenes + current
hera scene open res://Path.tscn              # open a scene
hera scene reload [res://Path.tscn]          # reload current/open scene from disk
hera scene save                              # save the edited scene
hera scene create res://Path.tscn [--root Node2D] [--force] [--open]
hera scene save-as res://Path.tscn [--force]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NotNull92/hera-agent-godot](https://github.com/NotNull92/hera-agent-godot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
