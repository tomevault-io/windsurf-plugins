---
trigger: always_on
description: validates the native-supplied event id before touching a path). Anything that
---

<!-- agentsync:managed memory-banner -->
> **Managed by [agentsync](https://agentsync.cc) — do not edit `AGENTS.md` directly.**
> To change it, edit `.agentsync/memory/AGENTS.md` (or the relevant
> `.agentsync/memory/fragments/*.md` fragment) and run `agentsync apply`.
> Direct edits here are reported as drift and overwritten on the next apply.
<!-- /agentsync:managed memory-banner -->

# CLAUDE.md

Project memory for Claude Code / agent sessions working on agentsync.

## What this is

agentsync is a single-machine Go CLI that centrally manages AI coding-agent
configurations (31 agents: nine deep adapters — Claude Code, OpenCode, Codex,
Cursor, Gemini CLI, Continue, Windsurf, Roo Code, Cline — plus a 22-agent generic
"breadth tier" for memory + MCP + skills). The
user keeps a canonical config in `~/.agentsync/` (small TOML + markdown,
committable to a dotfiles repo); `agentsync apply` renders it into each agent's
native config. It's bidirectional: native edits are detected as drift and merged
back via `reconcile`/`import`. Secrets are `${secret:…}`/`${env:…}` references
resolved at apply time from an age-encrypted vault.

**Read the docs before large changes:**
- [`docs/concepts.md`](docs/concepts.md) — the three-state model + every term.
- [`docs/architecture.md`](docs/architecture.md) — pipelines, drift classifier,
  secret invariants, package layering.
- [`docs/components.md`](docs/components.md) — package-by-package map.
- [`docs/capability-matrix.md`](docs/capability-matrix.md) — per-agent support.
- [`docs/superpowers/specs/2026-05-04-agentsync-design.md`](docs/superpowers/specs/2026-05-04-agentsync-design.md)
  — the authoritative v1.0 design. Note: a few items in its §"CLI surface" were
  aspirational and not wired in v1.0 (`apply --strict/--force/--agent` flags, an
  `agentsync skill` command) — trust the code over the spec on the CLI
  surface. Note the last of those has since SHIPPED: F5 added `skill list`
  alongside `subagent`/`command`/`hook`/`lsp list`.

## This repo is agentsync-managed — change `.agentsync/`, not the rendered files

agentsync dogfoods itself: this repository's own agent configuration is managed
by agentsync ([agentsync.cc](https://agentsync.cc)). The canonical source of
truth is the project-scope **`.agentsync/`** tree at the repo root — memory
(`.agentsync/memory/AGENTS.md`), skills (`.agentsync/skills/<name>/SKILL.md`),
MCP servers, subagents, commands, hooks, and `agentsync.toml`.

The native agent files are **rendered** from that tree by `agentsync apply
--scope project`; they are build output, not sources — do not hand-edit them:

- this file — `CLAUDE.md` (claude memory) and `AGENTS.md` (codex memory) — is a
  passthrough render of `.agentsync/memory/AGENTS.md`;
- `.claude/skills/…` and `.agents/skills/…` render from `.agentsync/skills/…`.

**To change memory, skills, MCP config, or any agent setting in this repo, edit
the canonical file under `.agentsync/` and re-render with `agentsync apply
--scope project`.** A direct edit to a rendered file is overwritten on the next
apply and surfaces as drift in `agentsync status --scope project`; to fold an
edit already made in a native file back into the canonical source, capture it
with `agentsync import`/`reconcile --scope project` instead.

## Keep the docs in sync — non-negotiable

Docs are part of the contract, not an afterthought. **No commit may change an
interface, a contract, the canonical schema, the CLI surface, or load-bearing
logic and leave the docs out of date.** If you change behavior, update the docs
in the *same* commit. A reviewer should never have to wonder whether the prose
or the code is the source of truth. Treat a stale doc as a bug.

When you touch… | …also update in the same commit
--- | ---
the `Adapter` interface / `DestWriter` / render or capture contracts | `docs/architecture.md` (§3–§5), `docs/components.md`
a CLI command, subcommand, or flag | `docs/user-guide.md` command reference, `README.md` quickstart, `website/src/content/docs/reference/cli.mdx`
agent/component coverage (a `Skip` goes native, a new adapter, a new component) | `docs/capability-matrix.md` (mirrored to the website), the matrices in `README.md` + `docs/user-guide.md`
the canonical schema / `~/.agentsync/` layout | `docs/concepts.md`, `docs/architecture.md` (§2), the layout block in `docs/user-guide.md`
the secret-handling invariants | the section below, `SECURITY.md`
anything user-visible | `CHANGELOG.md` (under `[Unreleased]`)

If a change makes a sentence in those docs false, the change is not done until
the sentence is fixed.

**The docs website (`website/`).** The published site at
[agentsync.cc](https://agentsync.cc) is an Astro Starlight project. Its four
**contract pages** (concepts, architecture, components, capability matrix) are
*generated* from `docs/*.md` by `website/scripts/sync-docs.mjs` at build time, so
they can never drift — never hand-edit the generated copies (they're gitignored).
The rest of the site (getting-started, guides, recipes, reference, FAQ) is
authored prose that is the source of truth for *itself*; when you change the CLI
surface or agent coverage, update the website pages listed in the table above in
the same commit, just like the `docs/` files. See `website/README.md`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spxrogers/agentsync](https://github.com/spxrogers/agentsync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
