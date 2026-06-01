---
trigger: always_on
description: This is the **elizaOS** monorepo: an open-source framework for building and
---

# elizaOS — repository guide for agents

This is the **elizaOS** monorepo: an open-source framework for building and
deploying autonomous AI agents, plus the runtime, CLI, dashboard, cloud
backend, native bridges, and first-party plugins built on top of it. The repo
is **self-contained** — everything needed to run, test, and ship an Eliza agent
lives here.

`CLAUDE.md` and `AGENTS.md` in every directory are **identical** — author
`CLAUDE.md`, then copy it to `AGENTS.md`. Read the package-local `CLAUDE.md`
before working inside any package or plugin; this root file is the map.

## Naming

Write **elizaOS** (not `ElizaOS`). npm scope is `@elizaos/*`. In plain language,
say **Eliza agents**. Exception: the **Eliza Classic** plugin keeps `Eliza`
(the 1966 chatbot it reimplements).

## Toolchain

- **Runtime:** [Bun](https://bun.sh) (`packageManager` is pinned in
  `package.json`) on **Node 24** (`engines.node`). ESM only (`"type": "module"`).
- **Monorepo:** [Turbo](https://turbo.build) drives `build` / `typecheck` /
  `lint` / `test` across workspaces. Workspace globs are in `package.json`
  (`packages/*`, `plugins/*`, `packages/native/*`, `packages/os/*`,
  `packages/examples/*`, `packages/cloud-services/*`, …).
- **Lint/format:** [Biome](https://biomejs.dev) (`biome.json`). Ignore globs in
  `.biomeignore`.
- **Tests:** Vitest, orchestrated by `packages/scripts/run-all-tests.mjs`.
- **TypeScript:** project-references build; root `tsconfig.json`,
  `tsconfig.base.json`, per-package `tsconfig.json`.

## Root commands

```bash
bun install            # workspace install (runs postinstall: submodules, patches)
bun run dev            # boot the API + dashboard UI (packages/app-core dev-ui)
bun run build          # turbo build across the workspace
bun run verify         # typecheck + lint (alias: bun run check) — run before "done"
bun run lint           # biome lint via turbo
bun run format         # biome format via turbo
bun run typecheck      # tsc across workspace (8 GB heap)
bun run test           # full suite (run-all-tests.mjs)
bun run test:server    # core/agent/app-core/shared/vault/elizaos/skills/scenario-runner
bun run test:client    # app/ui + lifeops/companion/training plugins
bun run test:e2e       # end-to-end lane
bun run start          # run an agent (packages/agent start)
bun run clean          # nuke dist/.turbo/node_modules, reinstall, rebuild
bun run cloud:mock     # boot the full local cloud stack with mocks
```

Scope any command to one package with `--cwd`:
`bun run --cwd packages/core test`. The repo has ~200 root scripts; the list
above is the day-to-day set. Use `bun run` with no args to print them all.

## Repo map — where to find what

```
packages/        framework, shared libraries, and product surfaces
  core/          @elizaos/core — runtime, types, agent loop, memory/state, model layer
  agent/         @elizaos/agent — AgentRuntime, plugin loader, default plugin map
  app-core/      API + dashboard host; dev/build orchestration (dev-ui.mjs)
  elizaos/       the `elizaos` CLI — create / info / upgrade / version; project + plugin templates
  prompts/       shared prompt scaffolding
  shared/        cross-package utilities + brand assets
  ui/            shared React component library
  app/           web + desktop dashboard (Vite + React; desktop shell)
  tui/           terminal UI
  skills/        runtime skills knowledge base (USE_SKILL)
  scenario-runner/ scenario + eval harness
  sweagent/      SWE-bench style coding-agent harness
  cloud-api/     managed backend API (Hono on Cloudflare Workers)
  cloud-frontend/ cloud dashboard (Vite + React) — see visual-review gate below
  cloud-shared/  shared cloud backend: db (Drizzle), billing, services, types
  cloud-sdk/ cloud-routing/ cloud-infra/  cloud client SDK, model routing, IaC
  contracts/     on-chain contracts + ABIs
  security/ vault/ soc2-verify/  secrets, key management, compliance tooling
  os/ os-homepage/ robot/        device/OS images, OS landing, robotics
  plugin-host-shim*/ plugin-worker-runtime/ plugin-remote-manifest/
                 plugin loading shims for native/electrobun/ios/android/worker targets
  homepage/ docs/ docs-elizacloud-redirect/  marketing site, docs site, redirects
  examples/      30+ standalone runnable examples (each has its own README)
  benchmarks/    30+ evaluation suites (each has its own README + harness)

plugins/         runtime plugins and app plugins
  plugin-<model>/      openai, anthropic, google-genai, groq, openrouter, xai, ollama, …
  plugin-<connector>/  discord, telegram, farcaster, slack, imessage, whatsapp, x, …
  plugin-native-*/     native device bridges (camera, contacts, calendar, location, …)
  plugin-local-inference/  on-device llama.cpp / omnivoice / whisper (git submodules under native/)
  plugin-sql/ plugin-localdb/ plugin-inmemorydb/  storage adapters
  plugin-companion/ plugin-documents/ plugin-lifeops/ plugin-health/ …  app plugins

scripts/         repo automation        patches/   dependency patches
skills/          runtime skill packages turbo.json knip.json  build + dead-code config
```

Every package and plugin carries its own `CLAUDE.md` / `AGENTS.md` (identical)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elizaOS/eliza](https://github.com/elizaOS/eliza) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
