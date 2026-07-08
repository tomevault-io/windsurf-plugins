---
trigger: always_on
description: Behavioral eval lab for superpowers. TypeScript, runs on Bun (≥1.3).
---

# Superpowers Evals

Behavioral eval lab for superpowers. TypeScript, runs on Bun (≥1.3).

The active runner is the Gauntlet-backed **Quorum**. Code, CLI, paths, and
inline prose all use lowercase `quorum`; the capitalized form `Quorum` appears
in headings and the actor table.

## Canonical Actors

Keep the actors straight; confusing them is the most common triage error.
These names are used everywhere: docs, CLI output, code, filenames, and commit
messages.

| Actor | What it is | Where it lives / its files |
|---|---|---|
| **Gauntlet** | General-purpose QA framework; the `gauntlet` CLI. A black-box tester. | repo `github.com/prime-radiant-inc/gauntlet`; on `PATH` as `gauntlet` (via `bun link` or `GAUNTLET_ROOT`) |
| **Gauntlet-Agent** | The LLM inside Gauntlet that drives the Coding-Agent and self-grades against the story's ACs. | model e.g. `claude-sonnet-4-6`; event stream -> `<run>/gauntlet-agent/results/<runId>/run.jsonl`; verdict -> `result.{json,md}` |
| **Coding-Agent** | The agent under test. Instances: **Claude**, **Codex**, **Antigravity**, **Gemini**, **Kimi**, **OpenCode**, **Pi**, and **Copilot**. | session log -> under the per-run throwaway `$HOME` (`<run>/home/<agent-config-subdir>/...`, e.g. `.claude`/`.codex`); files it writes -> `<run>/coding-agent-workdir/` |
| **Quorum** | The TypeScript wrapper. Owns setup, Coding-Agent adaptation, deterministic checks, and final verdict composition. | repo `superpowers-evals/src/`; `<run>/verdict.json` |

A run involves two LLMs: the **Gauntlet-Agent** (QA tester) and the
**Coding-Agent** (subject). Separate models, logs, and token costs.

## Commands

- **install**: `bun install`
- **test**: `bun test`
- **test single**: `bun test test/runner-unit.test.ts`
- **lint**: `bun run lint` (biome)
- **format**: `bun run format` (biome)
- **typecheck**: `bun run typecheck` (tsc --noEmit)
- **check (lint+typecheck+test)**: `bun run check`
- **validate scenarios**: `bun run quorum check`
- **run scenario**: `bun run quorum run scenarios/<name> --coding-agent <agent>`
- **run scenario with credential**: `bun run quorum run scenarios/<name> --coding-agent <agent> --credential <name>`
- **list scenarios**: `bun run quorum list`
- **scaffold scenario**: `bun run quorum new <name>`
- **show verdict**: `bun run quorum show [<target>]`
- **show costs**: `bun run quorum costs [<target>] [--with-gauntlet]`
- **run all**: `bun run quorum run-all [--coding-agents X,Y] [--jobs N]`
- **run all with credentials**: `bun run quorum run-all [--coding-agents X,Y] [--credentials cred1,cred2] [--jobs N]`
- **show batch**: `bun run quorum show <batch-id>` (matrix view)
- **grid-manifest**: `bun run quorum grid-manifest [--out <path>]` (emit the scenario × agent × os eligibility matrix the dashboard reads)
- **dashboard**: `bun run dashboard [--results <dir>] [--port N] [--manifest <path>]` (read-only web matrix over results/ + grid-manifest.json)
- **shared appliance design/runbook**: `docs/superpowers/specs/2026-06-18-shared-eval-appliance-design.md` and `docs/appliance-runbook.md`

Per-coding-agent config: `coding-agents/<name>.yaml`. Per-coding-agent HOWTO:
`coding-agents/<name>-context/HOWTO.md`. Each agent seeds its config under the
per-run throwaway `$HOME` (e.g. `<runDir>/home/.claude` /
`<runDir>/home/.codex`) at provision time; no committed home skeleton ships.
Spec: `docs/superpowers/specs/2026-05-22-harness-model-design.md`.

## Architecture

- `src/runner/` — per-run orchestration: setup, pre-checks, Gauntlet drive, capture, post-checks, verdict (`index.ts`); `context.ts`, `phase.ts`, `errors.ts` (staged `RunError`), `stopped.ts`.
- `src/checks/` — sources `prelude.sh` (the bare-verb DSL: one shell function per FS check verb, plus `not`/`check-transcript`/`setup-helpers`, each delegating to the TS CLIs) then `checks.sh`, runs `pre()`/`post()`, collects structured check records (the verb functions emit them to `QUORUM_RECORD_SINK`). A crash-band verb exit (126/127/signal) anywhere in a phase aborts the phase (`set -E` ERR trap), so a broken check can never vanish from the verdict.
- `src/composer.ts` — composes Gauntlet-Agent verdict + deterministic checks into `pass | fail | indeterminate`.
- `src/contracts/` — zod schemas + types: `verdict.ts` (the `verdict.json` shape), `agent-config.ts`, `batch.ts`, `economics.ts`, `gauntlet.ts`, `credential.ts` (CredentialSchema — `model`, `api`, `base_url`, `auth`, `api_key_env`, `harnesses`, `max_concurrency`, `launch_spacing_seconds`, `os_support`, `compat`).
- `src/credentials/` — credential loading (`index.ts`), resolution (`resolve.ts`: `resolveCredentialNameForAgent`, `resolveApiKey`, `limiterKey`), and `quorum check` validation (`check.ts`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prime-radiant-inc/superpowers-evals](https://github.com/prime-radiant-inc/superpowers-evals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
