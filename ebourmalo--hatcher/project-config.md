---
trigger: always_on
description: Guidance for AI agents working on this repository.
---

# AGENTS.md

Guidance for AI agents working on this repository.

## Project Summary

Hatcher is a TypeScript CLI that deploys local AI agent runtimes to cloud sandbox providers.

Supported runtimes:
- `hermes`
- `claude-code`

Supported providers:
- `cloudflare`
- `daytona`

Core contract:
- `hatcher.json` is the source of truth for agent identity, runtime, model, skills, provider, and `deployment.env`.
- `identity.file` is the runtime identity/persona file staged into the image, for example `CLAUDE.md` for Claude Code or `SOUL.md` for Hermes.
- `.hatcher/` is generated deployment output. Do not treat it as source unless debugging a generated artifact.
- `.hatcher/state.json` stores the current deployment/session state.
- Secrets must never be written into `hatcher.json`.

## Common Commands

Use these from the repository root:

```bash
npm run typecheck
npm run build
npm test
npm run coverage
```

Focused tests are preferred while iterating:

```bash
npm test -- tests/docker/cloudflare-worker.test.ts
npm test -- tests/cli/cloudflare-terminal.test.ts
npm test -- tests/tui/operations.test.tsx
npm test -- tests/sandboxes/provider-contract.test.ts
npm test -- tests/e2e/deploy.matrix.test.ts
```

Optional real-local Hermes smoke test:

```bash
npm run test:local-hermes
```

Only run provider commands such as `wrangler deploy`, `wrangler delete`, `daytona sandbox create`, or raw `hatcher deploy` when explicitly requested. They can mutate remote infrastructure and cost real money.

`hatcher test image` builds a local Docker image and runs a readiness smoke test. It does not deploy, but it can be slow and requires Docker, so run it only when image behavior is relevant or explicitly requested.

For changes that touch deploy / probe / destroy semantics, prefer the
end-to-end harness over ad-hoc `hatcher deploy` runs:

```bash
npm run e2e -- --combo claude-code:cloudflare
npm run e2e -- --all
```

The harness lives at `tools/e2e/` (see `tools/e2e/README.md` for the
full lifecycle and flags). Every run writes a `report.json` per cell
plus a `summary_report.json` for the run under `tools/e2e/runs/`, so
you can answer "did it pass and which assertions failed?" with one
`jq` call instead of re-deploying to scrape stdout. Use `--keep` to
retain failed sandboxes for live debugging, and always tear them
down with `hatcher destroy` from the retained cwd when done.

## Repository Map

- `src/cli/`: command implementations and CLI entrypoint.
- `src/tui/`: Ink-based TUI components and operation runner.
- `src/agents/`: runtime detection and runtime-specific local config parsing.
- `src/sandboxes/`: provider implementations for deploy, destroy, connect, and validation.
- `src/docker/`: Dockerfile template resolution, generated Cloudflare Worker source, and `.hatcher/` artifact generation.
- `src/schema/`: `hatcher.json` Zod schema.
- `src/session/`: local deployment state management.
- `docker/`: Dockerfile templates copied into generated deployment contexts.
- `tests/`: unit, integration, e2e, and in-container tests.

## CLI UX Rules

The CLI uses Ink, not `pi-tui`.

For commands other than no-op/help flows:
- Show what the command will do before executing.
- Ask one top-level confirmation unless `--yes` is passed.
- Show a persistent operation list with status markers.
- External commands must be visible before confirmation.
- `--verbose` streams command output below the operation list.
- Non-TTY fallback must still show static operation labels and statuses.

Operation markers:
- pending: `·`
- running: spinner
- succeed: `✓`
- failed: `✗`

Do not reintroduce per-command prompts like `Run: npm install ...?`. External commands are authorized by the single top-level confirmation.

## Deployment Model

`hatcher deploy` should:
1. Load and validate `hatcher.json`.
2. Sync runtime config into `hatcher.json` when needed.
3. Resolve env vars listed in `deployment.env`.
4. Regenerate `.hatcher/`.
5. Check provider auth and existing deployment state.
6. Deploy or update through the provider.
7. Record `.hatcher/state.json`.
8. Validate remote access when supported.

`deployment.env` is the single source of truth for sandbox env injection.

Runtime file mappings live in `src/agents/*` through `RuntimeConfig`. The expected file flow is:

```text
runtime-defined source paths -> .hatcher/ staging -> Docker image runtime paths
```

Dockerfiles must only copy from `.hatcher/`. Do not make Docker templates depend on files in the original runtime source folder or repository root. Required runtime files must fail with explicit errors when missing; optional runtime files may be skipped.

Claude Code runtime files are sourced from `~/.claude` and `~/.claude.json` by default. Hermes runtime files are sourced from `~/.hermes` by default. Optional bundles (e.g. Hermes `~/.hermes/auth.json` for OAuth-only models) are declared in the runtime's `additionalFiles[]` and copied into `.hatcher/` only when the host file is present — never required, never failing.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ebourmalo/hatcher](https://github.com/ebourmalo/hatcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
