---
trigger: always_on
description: This file is for **any agent (human or LLM) working on the phantombot codebase itself**. If you're a Claude Code session, an OpenAI-Codex agent, a future maintainer, or just yourself catching up after time away — read this before opening a PR. It captures the small set of project-specific conventions that aren't obvious from the code.
---

# AGENTS.md — guide for any agent contributing to phantombot

This file is for **any agent (human or LLM) working on the phantombot codebase itself**. If you're a Claude Code session, an OpenAI-Codex agent, a future maintainer, or just yourself catching up after time away — read this before opening a PR. It captures the small set of project-specific conventions that aren't obvious from the code.

> **Naming-collision warning.** This `/AGENTS.md` (project root) is **not the same as** `personas/<name>/AGENTS.md` (a persona-specific tool-hint file phantombot reads when loading that persona). They share a name because OpenClaw uses the same convention for persona files; phantombot's persona loader accepts both `tools.md` and `AGENTS.md` as the persona-tools slot. If a contributor refers to "AGENTS.md" without context, they almost always mean *this* file (the repo-level one). If you're editing a file under `personas/` or `agents/`, you're touching a persona, not the contributor guide.

## The contributing discipline (READ FIRST)

**Every PR that changes user-facing behavior, architecture, or developer workflow must update both `README.md` and this file in the same PR.** No exceptions. Reviewers should reject PRs where the docs don't match the code.

What "user-facing behavior" means:
- New / removed / renamed CLI subcommand or flag
- Change to systemd unit shape (added/removed timer, new `EnvironmentFile=`, etc.)
- Change to where files live (config path, memory db path, persona dir, lockfile)
- Change to credential discovery / hygiene rules
- Change to release pipeline (asset names, version scheme, trigger)
- Change to architecture invariants (single persona at runtime, runLock, etc.)

What's *not* user-facing and doesn't need a docs update:
- Pure refactors that preserve behavior
- Bug fixes that restore the documented behavior
- Internal lib changes with no public API change
- Tests

If unsure, update the docs. Cheaper than the post-merge "wait, the README says X but the code does Y" archeology.

## Build / test / typecheck

```bash
~/.bun/bin/bun install                  # install deps (cron-parser, citty, smol-toml, @clack/prompts)
~/.bun/bin/bun tsc --noEmit             # typecheck
~/.bun/bin/bun test                     # full test suite
~/.bun/bin/bun test tests/lib-X.test.ts # one file
~/.bun/bin/bun run build                # → dist/phantombot (linux x64-baseline; ~98 MB)
~/.bun/bin/bun run build:arm64          # → dist/phantombot-arm64 (cross-compile)
```

`bun-version` is pinned to `1.x` in CI for reproducibility (see `.github/workflows/release.yml`).

The build target **must remain `bun-linux-x64-baseline`** (not plain `bun-linux-x64`). The supervisor box that runs kai is pre-AVX2 silicon; the non-baseline binary SIGILLs on launch there. If you "optimise" to plain x64, you'll break production. See PR #37 for the post-mortem.

## Repo layout

```
phantombot/
├── README.md                 # user-facing docs
├── AGENTS.md                 # ← this file
├── docs/
│   ├── architecture.md
│   └── adding-a-harness.md
├── .github/workflows/release.yml   # auto-release per merged PR
├── package.json
├── bunfig.toml
├── tsconfig.json
├── src/
│   ├── index.ts              # entry point: runs Citty dispatcher
│   ├── version.ts            # CI sed-replaces "0.1.0-dev" with "1.0.<PR_NUMBER>"
│   ├── config.ts             # XDG paths + TOML loader (env vars > config > defaults)
│   ├── state.ts              # phantombot-managed state (default persona)
│   ├── persona/
│   │   ├── loader.ts         # accepts BOOT.md / SOUL.md / IDENTITY.md, MEMORY.md, tools.md / AGENTS.md
│   │   └── builder.ts        # buildSystemPrompt + MEMORY_TOOLS_SECTION + CREDENTIALS_SECTION
│   ├── memory/
│   │   └── store.ts          # bun:sqlite store: turns table + capture_log table
│   │                         #   (capture_log backs the 30-turn nudge + doctor)
│   ├── importer/
│   │   └── openclaw.ts       # OpenClaw → phantombot persona import
│   ├── orchestrator/
│   │   ├── turn.ts           # one-turn coordinator (persona → memory → screen → harness → persist)
│   │   ├── fallback.ts       # harness chain (primary → fallback)
│   │   ├── screen.ts         # makeScreener: threat-screen wiring for UNTRUSTED turns (see "Security perimeter")
│   │   ├── retrieval.ts      # makeRetriever: semantic recall of prior turns/memory for the prompt
│   │   ├── recovery.ts       # generateRecoveryReply: graceful user-facing message on harness failure
│   │   └── turnIndexer.ts    # makeTurnIndexer: embeds persisted turns for later retrieval
│   ├── channels/             # channel-agnostic core + per-channel adapters (see "Channel layer")
│   │   ├── core/
│   │   │   ├── types.ts      # Channel / ChannelTransport / ChannelMessage + capabilities + encrypt/decrypt seam
│   │   │   ├── engine.ts     # the streaming turn engine + server loop (channel-blind)
│   │   │   ├── routing.ts    # group-reply decision logic (pure)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phantomyard/phantombot](https://github.com/phantomyard/phantombot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
