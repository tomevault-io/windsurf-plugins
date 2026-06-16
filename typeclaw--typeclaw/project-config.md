---
trigger: always_on
description: > **Audience.** This file is for the AI assistant (or human contributor) working on the typeclaw source tree — the **dev stage** in `## Stages` below. It is **NOT** the runtime prompt for typeclaw agents; that prompt is composed in `src/agent/index.ts` via `composeSystemPrompt` and can be dumped with `bun run debug:prompt`. When sections below describe runtime behavior, they describe what the code in `src/` does — not instructions to the runtime agent.
---

# Agent Guidelines

> **Audience.** This file is for the AI assistant (or human contributor) working on the typeclaw source tree — the **dev stage** in `## Stages` below. It is **NOT** the runtime prompt for typeclaw agents; that prompt is composed in `src/agent/index.ts` via `composeSystemPrompt` and can be dumped with `bun run debug:prompt`. When sections below describe runtime behavior, they describe what the code in `src/` does — not instructions to the runtime agent.

## Default scope

If the user asks something, it's always about the typeclaw project itself until they specify another scope. Don't drift into upstream/downstream projects (agent-messenger, plugins consumed via npm, etc.) just because the conversation mentions them.

Write only inside this repo and pre-approved temp dirs (`/tmp/`, `$TMPDIR`). Never touch the user's global skills, configs, or agent identities (`~/.claude/`, `~/.agents/`, `~/.config/opencode/`), typeclaw runtime state (`~/.typeclaw/`), credentials (`~/.ssh/`, `~/.aws/`, etc.), shell/OS config, or sibling repos. If you think you need to, stop and ask.

## Pre-commit checks

Before every commit, all three must pass:

```sh
bun run typecheck
bun run lint
bun run format
```

No exceptions. No `--no-verify`. No partial fixes.

## Debugging the system prompt

`bun run debug:prompt` dumps the rendered system prompt for each session-origin kind (`tui`, `cron`, `channel`, `subagent`) with placeholder values, plus a per-section token/char/byte breakdown.

```sh
bun run debug:prompt                       # all 4 origins
bun run debug:prompt --origin cron         # just one
bun run debug:prompt --origin channel --no-git-nudge
```

`composeSystemPrompt` (`src/agent/index.ts`) is the right entry point if you're adding a new section. The cache-suffix contract (least-volatile first → identity → runtime → origin+role → git → memory → now) is enforced by both the helper and `scripts/dump-system-prompt.test.ts`. Reorder one without the other and CI fails. The trailing `## Now` block is pinned last to keep the cache prefix stable across sessions — don't move it.

When `memory.vector.enabled` is true the `# Memory` section is omitted from the system prompt entirely (`createSession`'s `suppressSystemMemory`, derived once at boot from the restart-required vector flag in `src/run/index.ts`). Vector agents inject long-term memory **per-turn into the user prompt** instead — the memory plugin's `session.turn.start` hook renders all shards (under budget) or top-K hybrid-search results (over budget) into `event.retrievalContext.results`, which the four turn-drivers (server TUI, channel router, cron consumer, subagent runner) append to the user text. The invariant `suppressSystemMemory === memory.vector.enabled` is what prevents double-injection; a session must never carry memory in both places.

The other half of the memory loop is **consolidation**: the `dreaming` subagent (`src/bundled-plugins/memory/dreaming.ts`, cron `memory.dreaming.schedule`, default `*/30 * * * *`) reads undreamed daily-stream fragments and rebalances them into `memory/topics/<slug>.md` shards. Three load-bearing invariants make a bad LLM run non-destructive: the **citation-superset check** (every previously-cited fragment id must still be cited after the run, in `fragments:` or `superseded:`, else the whole run reverts via `restoreShardSnapshot`), **runtime-owned frontmatter** (`cites`/`days`/`lastReinforced` are recomputed from citations every run — the subagent never sets them), and **fragment-GC gating** (`compactDailyStreams` drops dreamed-and-uncited fragments only when shards were actually rewritten this run, never on stale citations). Dreamed-ids advance even on a citation-superset revert — the conscious anti-loop tradeoff. See [/docs/internals/memory](https://typeclaw.dev/docs/internals/memory).

Slim vs full mode is decided by `deriveSystemPromptMode` (exhaustive `switch` on `origin.kind`). `tui` and `channel` get the full operator-facing prompt; `cron` and `subagent` get the slim base (~245 tok). Production subagents bypass the slim base entirely via `systemPromptOverride`; the slim path only fires for cron today.

## Release

Use the **Release** GitHub Actions workflow (`workflow_dispatch`, see `.github/workflows/release.yml`). It validates the version, runs checks, bumps `package.json`, builds and pushes multi-arch base to `ghcr.io/typeclaw/typeclaw-base:X.Y.Z`, verifies cross-platform pullability, publishes to npm with provenance, then tags + releases. Tags have no `v` prefix.

The workflow is the only supported release path. The GHCR-first-then-npm ordering is load-bearing for the version-pin invariant: a user who `npm install`s before the base image lands cannot `typeclaw start`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [typeclaw/typeclaw](https://github.com/typeclaw/typeclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
