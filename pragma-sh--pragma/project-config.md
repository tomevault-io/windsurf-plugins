---
trigger: always_on
description: > This file is the single source of truth for repo-wide rules. `CLAUDE.md` is a symlink
---

# Pragma — Agent & Contributor Guide

> This file is the single source of truth for repo-wide rules. `CLAUDE.md` is a symlink
> to it. Each package/app/crate has its own `AGENTS.md` with deeper specifics — read the
> relevant one before touching that area.

## What is Pragma?

Pragma is a **macOS + Linux + Windows desktop app** (Tauri v2) moving toward a
host-server + thin native-client architecture. The host runs persistent, worktree-scoped
terminal sessions through `pragma-server`; native clients connect over a local Unix
socket (on all three platforms), or over a bridge that presents a remote SSH host or a
WSL distribution as one. Plugins for opencode, Claude Code, and Cursor report status through
the `pragma-cli` helper.

## North star: clean, reusable, consistent

The overriding priority is a **clean, reusable architecture** with **consistent rules
across TypeScript and Rust**. When you write code:

- **Reuse before you write.** Search for an existing helper, component, or constant
  first. Lift duplicated logic into a shared location the moment it appears twice.
- **Do not be afraid to create a new package or directory.** If something is shared,
  or _could_ be shared, give it a home (`packages/*`). If a value is referenced in
  more than one place, it belongs in `@pragma-sh/constants`, not inline. Small,
  single-purpose packages are encouraged.
- **One source of truth.** Never copy a value across the TS/Rust boundary by hand —
  put it in `@pragma-sh/constants` (see `packages/constants/AGENTS.md`).
- **Keep the two languages in lockstep.** The same concept should be named, layered,
  and error-handled the same way in TypeScript and Rust. See _Code standards_ below.
- **Suggest sweeping changes.** If you see a cleaner structure,
  propose and make it — restructuring for clarity is welcome, not discouraged.
- **Host-tool plugins stay out of core — ask first.** A host-tool plugin/agent package
  (`packages/*-plugin`, such as opencode/Claude/Cursor integrations) is
  **self-contained data plus its own bundled assets**. It must **not** add or modify code
  in pragma core (`apps/pragma`, including `src-tauri`), the server
  (`crates/pragma-server`), the UI, the CLI (`crates/pragma-cli`), or the SDK
  (`packages/sdk`) **without explicit owner permission**. A host-tool plugin installs
  itself through its host tool's own plugin mechanism, never through per-plugin Pragma
  code. Launchable Pragma agents are contributed through the `@pragma-sh/plugin`
  `defineAgent` API (or the built-in Pragma plugin), not per-tool JSON files copied by
  core. There are deliberately **no** per-tool core installers: the old
  `opencode_plugin.rs` / `claude_plugin.rs` installers were **removed**. The generic
  Pragma plugin runtime is the exception: infrastructure for `.pragma/config.json` plugins lives in
  `packages/plugin`, `apps/pragma/src/plugins`, and `apps/pragma/src-tauri/src/plugins.rs`.

## Keeping this guide current (self-improvement)

**This document is living. If it is wrong, stale, or incomplete, fix it as part of your
change — that is expected, not optional.** A guide that drifts from reality is worse
than no guide.

- **Edit the relevant AGENTS.md in the same change that makes it outdated.** Add a
  package, move a file, change a command, bump a tool, adopt a new pattern → update the
  matching AGENTS.md (root and/or child) in the same commit. Because `CLAUDE.md` is a
  symlink to the root AGENTS.md, both humans and agents stay in sync automatically.
- **Mirror it in the skills.** Canonical user-facing skill sources live under `skills/`
  and are symlinked into `.agents/skills/` (which `.claude/skills` also exposes). Internal
  contributor skills live directly in `.agents/skills/`, so that directory contains both
  internal and user-facing skills. If you
  change a workflow here, update the relevant skill (`pragma-architecture`,
  `shared-constants`, `tauri-command`, `code-quality`, `pragma`) too, and
  add a new skill when you add a substantial new workflow.
- **Ship the website with the feature.** A user-visible change is not done until
  `apps/www` matches it: add or update the `/docs` page (and its `meta.json` entry), the
  landing-page copy or bento card when the feature is worth announcing, and any page the
  change makes wrong — wiki, CLI, SDK, keybindings, disk layout. Same change, same commit,
  exactly like the AGENTS.md rule above. A feature nobody can read about does not exist.
- **When you discover something the hard way, write it down.** A non-obvious gotcha, a
  setup step, a "don't do X because Y" — capture it here (or in the relevant child
  AGENTS.md) so the next person (or agent) doesn't rediscover it.
- **Prefer fixing the guide over working around it.**
- **Keep it concise.** Prune advice that no longer applies. Length is not authority;
  accuracy is.

## Tech stack

| Concern           | Choice                                                                                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pragma-sh/pragma](https://github.com/pragma-sh/pragma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
