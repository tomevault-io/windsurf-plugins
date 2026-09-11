---
trigger: always_on
description: DeepSeek Harness is an all-plugin Cordis agent harness. Read [docs/architecture.md](docs/architecture.md) before changing `packages/`; follow [docs/AGENTS.md](docs/AGENTS.md) for documentation.
---

# AGENTS.md

DeepSeek Harness is an all-plugin Cordis agent harness. Read [docs/architecture.md](docs/architecture.md) before changing `packages/`; follow [docs/AGENTS.md](docs/AGENTS.md) for documentation.

## Pre-release stance: foundation over blast radius

**Remove at the first tagged release.** Until then, prefer correct foundations to compatibility shims: rename or repackage freely and update every reference. Backends reject old on-disk formats. SQLite uses monotonic `SCHEMA_VERSION`; `dsh-session` keeps `SESSION_FORMAT_VERSION` at `0` with no compatibility promise.

**Application launch.** Only `dsh` profiles launch supported Node apps; package bins, demos, and public SDK argv escapes are forbidden ([rule](docs/architecture.md#application-launch)).

## Repository layout

```
vendor/      Vendored Cordis source — manifest + sync procedure in vendor/README.md
packages/    @deepseek-ai/dsh-<pkg> workspaces at packages/<group>/<pkg>/
  core/        product API spine: session, system-prompt, tools, agent, agent-loop
  api/         Remote BFF assembly and Typert RPC gateway
  typert/      type graph generator, loader, and runtime registry
  llm/         LLM capability: Service Definition/Consumer + DeepSeek providers
  e2b/         E2B POC: sandbox + FS/subprocess adapters
  shell/        bash capability: Service Definition + local/pwsh providers + shell Consumers
  subprocess/  subprocess capability + local process-tree provider + shared Win32 library
  terminal/         persistent sessions
  fs/          filesystem capability + policy
  lsp/         language-server capability
  skill/       skill provider registry + local impl + catalog/loader tool
  web/         web capability: Service Definition + search/fetch providers + tool Consumer
  compaction/     compaction capability + basic provider
  context/     request-context plugins
  subagent/    subagent capability: Service Definition + providers + delegation Consumers
  bundle/      installable dsh --profile patch-layer bundles
  workflow/    workflow capability + worker-thread provider + tool Consumer
  webhook/     webhook ingress
  todo/        todo_write tool
  plan/        plan mode as logged state
  preset/      per-session agent composition from preset cordis.yml files
  guard/       loop-hygiene + tool-timeout plugins
  self-modification/  the agent inspects/mounts its own plugins
  hooks/       Claude Code/Codex hook bridges + wire-protocol library
  session/     durable session data: persistence, projection, titles, telemetry
  identity/    anonymous identity
  settings/    user-settings capability + file provider
  credentials/ credential/authorization capabilities + env/.env provider
  acp/         automation-only Agent Client Protocol server
  interaction/ approval/interaction capabilities, permission, commands, ask-user
  boot/        shared profile/application boot glue
  sdk/         JSON-RPC protocol + TypeScript client/server
  examples/    reusable composition bundles (agent-spine)
  experimental/ private prototypes excluded from official releases
  support/     dev/test infrastructure
  util/        zero-dependency utilities
python/      Python SDK and bundled runtime (see python/README.md)
native/      @deepseek-ai/node-addon-landlock-run source of record (see native/README.md)
.agents/     Agent workflows and Agent Notes (`notes/`)
docs/        architecture, generated catalogs, postmortems, cookbook (see docs/AGENTS.md)
scripts/     repo gates and generators
website/     VitePress projection of selected bilingual docs/ sources
```

Package groups: [packages/README.md](packages/README.md).

## Commands

```sh
pnpm install            # pnpm workspaces, node ^22.19 || >=24
pnpm run clean           # remove build outputs and safe residue from deleted packages
pnpm run test           # unit tests
pnpm run test:coverage  # CI coverage gate: per-file 100% on packages/*/*/src
pnpm run test:e2e       # real-API tests; self-skip without DEEPSEEK_API_KEY
pnpm run test:expected  # owner-local process expectations
pnpm run test:snapshot  # keyless recorded-session replay through shipped profiles; filter: -t <name>
pnpm run test:snapshot:record  # re-record expected outputs (needs key)
pnpm run typecheck
pnpm run lint
pnpm run duplication    # cross-file TypeScript clone detection
pnpm run build          # tsc emits lib/types, tsdown bundles runtime
pnpm run hygiene        # knip + publint + workspace constraints + NodeNext consumer check
pnpm run check:windows-wine  # ONLY when diagnosing a known Windows failure (needs wine); CI owns this signal
pnpm run doc-sync       # all documentation gates; leaf list in scripts/run-gates.ts
pnpm run test:docs      # quick documentation checks (no build; doc-quick aggregate)
pnpm run website:build  # VitePress build (doubles as dead-link check)
pnpm dsh --profile headless "task"  # run one task from source (needs DEEPSEEK_API_KEY)
pnpm run demo:ptc -- "task"  # headless PTC mode run (needs key)
```

### Host sandbox failures


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [harshith397/sih](https://github.com/harshith397/sih) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
