---
trigger: always_on
description: This is the single source of truth for AI coding agents (Claude Code, Cursor, OpenCode, Codex, and similar) working in this repository. `CLAUDE.md` points here; do not duplicate hard constraints into per-tool files. Human contributors: also see [CONTRIBUTING.md](CONTRIBUTING.md).
---

# AGENTS.md — canonical agent context for PurrCode

This is the single source of truth for AI coding agents (Claude Code, Cursor, OpenCode, Codex, and similar) working in this repository. `CLAUDE.md` points here; do not duplicate hard constraints into per-tool files. Human contributors: also see [CONTRIBUTING.md](CONTRIBUTING.md).

## What this project is

PurrCode is an open-source Claude Code addon that plays a continuous cat-purring sound while Claude is processing a turn. Full spec: [docs/PurrCode-PRD-v0.4.md](docs/PurrCode-PRD-v0.4.md) — the PRD is the roadmap; amendments to it are PRs.

## Architecture map

```text
Claude Code (plugin hooks, async exec form)
        │  stdin JSON → argv start|stop|failure|session-end
        ▼
  hook adapter (`packages/cli` → dist/hook.js)
        │  control pipe (UDS / named pipe)
        ▼
  per-user player daemon (session registry, watchdog, fades)
        │
        ▼
  audio backend tiers: sidecar (1) → mpv (2) → native (3) → fake (CI)
```

Plugin package lives under `packages/plugin/marketplace/` (assembled via `pnpm plugin:build`). Persistent config / WAV cache: `${CLAUDE_PLUGIN_DATA}` when running as a plugin.

## Current state & how to pick work

The project is being built sprint-by-sprint from a written backlog. **Before doing anything, read [docs/planning/START-HERE.md](docs/planning/START-HERE.md)** — it has the progress board, dependency order, and the per-story workflow. Never start a story whose dependencies aren't done, and never cross a 🚧 human gate.

- Sprint plan and working agreements: `docs/planning/00-sprint-plan.md`
- Story details: `docs/planning/sprint-N-*.md` (acceptance checkboxes are the definition of done)
- Spike/architecture decisions: `docs/decisions/` (use `TEMPLATE.md`)

## Invoking the PurrCode agent

The callable project agent is **`purrcode`**. Canonical prompt: [docs/agents/purrcode.md](docs/agents/purrcode.md). Tool wrappers (frontmatter only) live at:

| Tool | Invoke | Wrapper |
| --- | --- | --- |
| **Cursor** | `/purrcode` or Task → purrcode | [`.cursor/agents/purrcode.md`](.cursor/agents/purrcode.md) |
| **Claude Code** | `@purrcode` or ask to use the purrcode agent | [`.claude/agents/purrcode.md`](.claude/agents/purrcode.md) |
| **OpenCode** | Tab to `purrcode`, or `@purrcode` | [`.opencode/agents/purrcode.md`](.opencode/agents/purrcode.md) |

Workflow skills (implement story / spike / sprint review) live under [`.cursor/skills/`](.cursor/skills/); Claude mirrors under [`.claude/skills/`](.claude/skills/) point at those files. OpenCode loads `AGENTS.md` plus [opencode.json](opencode.json) instructions.

## Commands

```bash
pnpm install                 # setup (Node 20+, pnpm workspace)
pnpm lint                    # ESLint + Prettier check
pnpm typecheck               # tsc --noEmit, strict
pnpm test                    # Vitest unit tests
PURRCODE_FAKE_AUDIO=1 pnpm e2e:fake-audio   # integration suite (fake backend)
pnpm sidecar:fetch-vendor    # pin miniaudio + stb_vorbis into packages/sidecar/vendor/
pnpm sidecar:build           # build host-platform purr-sidecar into packages/sidecar-<os>-<cpu>/
pnpm e2e:sidecar             # real sidecar lifecycle (needs a built binary; skipped if absent)
pnpm plugin:build            # assemble + validate Claude Code plugin (packages/plugin/)
pnpm --filter purr-code build  # compile CLI / hook / daemon → packages/cli/dist
```

MVP CLI (after build): `purrcode install|uninstall`, `on|off`, `status`, `volume`, `preview`, `doctor`. Ops notes: [docs/troubleshooting.md](docs/troubleshooting.md).

## Verifying changes headlessly

Set `PURRCODE_FAKE_AUDIO=1` (as in the Commands block above) to use the fake audio backend (story PC-101): it implements the `AudioBackend` interface by writing timestamped playback events (start/volume/fade/stop) to a JSONL log instead of a sound card. Vitest runs TypeScript tests directly — you do **not** need `pnpm --filter purr-code build` before `pnpm test` / `pnpm e2e:fake-audio`. Build is required for the real CLI binary, plugin assemble, and Claude Code install smokes. Perceptual audio QA (seams, pops, fatigue) is human-only; never claim to have verified it.

## Hard constraints (violations are P0)

1. **Fail silent, fail open.** Hook-path code must always exit 0 and never write a byte to stdout/stderr — a single transcript error notice in Claude Code is a critical bug. Errors go to the opt-in debug log only.
2. **No network code anywhere.** Runtime makes zero network requests. No telemetry, no analytics, no update checks.
3. **No new runtime dependencies** without a linked issue approving them. Dev dependencies need a reason in the PR.
4. **All child processes in exec form** (argument arrays). Never shell-interpolated strings, never `eval`, never `npx` in anything a hook executes.
5. **Treat hook stdin as untrusted:** schema-validate, size-cap, never interpolate its fields into shell commands or paths.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MathoAvito/purr-code](https://github.com/MathoAvito/purr-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
