---
trigger: always_on
description: description: Repository Context for AI Coding Assistants
---

---
description: Repository Context for AI Coding Assistants
audience: AI Assistants, LLMs, Autonomous Agents
---

# AGENTS.md — Repository Context for AI Coding Assistants

This document is the authoritative context file for any AI assistant, LLM, or autonomous agent operating within this repository. Read it in full before making any changes.

---

## Repository Identity

**Package name:** `fluksos`  
**npm command:** `npx fluksos@latest`  
**Type:** CLI scaffold generator — not an application, not a library  
**Audience:** Two distinct groups (documented below)

---

## Two Audiences

### End Users (CLI consumers)

End users run `npx fluksos@latest init nextjs my-app ./my-app --tier 3` and expect a production-ready project output. They never see the internals of this repository. Their only interface is the CLI.

From their perspective:
- The CLI is the product.
- The generated project is the output.
- Correctness means: the scaffold runs without errors, all files are present, dependencies install cleanly, and `pnpm dev` starts the app.

### Contributors and Developers

Contributors work inside this repository — they modify scripts, templates, validators, or add new stacks. This is their context document.

From their perspective:
- The CLI is a dispatcher (`bin/cli.js`).
- The stack scripts are the core logic (`stacks/<stack>/scripts/`).
- The templates are the static file sources (`stacks/<stack>/templates/`).
- The tests validate behavior without running the full scaffold.
- Structural changes require a planning workflow (see Slash Commands below).

---

## Repository Structure (What Matters)

The only directories that ship to npm are `bin/` and `stacks/`. All other directories are development tooling.

```
fluksos/
├── bin/
│   └── cli.js                    # CLI entry point — dispatches to stack scripts
├── stacks/
│   └── nextjs/
│       ├── scripts/
│       │   ├── init_project.js            # Full scaffold pipeline (13 steps)
│       │   ├── generate_action.js         # Server Action code generator
│       │   ├── generate_rpc_hook.js       # Hono RPC hook generator
│       │   ├── validate-architecture.js   # AST + filesystem architectural rules
│       │   ├── validate-config.js         # Configuration correctness rules
│       │   ├── validate-ui-state.js       # UI boundary rules
│       │   └── lib/
│       │       ├── ast-parser.js          # Core AST rule engine
│       │       └── report-generator.js    # Violation report formatter
│       ├── templates/
│       │   ├── root/                      # Workspace-level config (biome.json, lefthook.yml, .github/workflows)
│       │   ├── app-common/                # Always applied — base Next.js files
│       │   ├── app-tier-2/                # Applied for --tier 2 and --tier 3
│       │   │   └── src/
│       │   │       └── lib/
│       │   │           ├── safe-action.ts # Rate-limited server actions
│       │   │           └── rate-limit.ts  # Upstash Redis limiter
│       │   ├── app-tier-3/                # Applied for --tier 3 only
│       │   │   ├── Dockerfile             # Standalone production build
│       │   │   └── docker-compose.yml     # Local pgvector database
│       │   ├── observability/             # Docker Compose, OTel, Grafana Faro, Loki, Tempo, Pyroscope
│       │   └── tests/                     # K6 performance test scripts
│       ├── tests/
│       │   ├── mock-project/              # Fixture project for unit tests (not shipped)
│       │   ├── validate-architecture.test.js
│       │   ├── validate-config.test.js
│       │   └── validate-ui-state.test.js
│       └── README.md                      # Stack-specific developer notes
├── package.json                           # npm package — bin entry, files field
├── README.md                              # Public documentation
└── AGENTS.md                              # This file
```
---

## How the CLI Works

`bin/cli.js` is the sole entry point. It:

1. Prints the ASCII logo on every invocation.
2. Parses the first positional argument as a command (`init`, `generate`, `validate`).
3. Looks up the command's target in `STACK_REGISTRY` — an in-file object that maps stack names to their script files, generators, validators, and tiers.
4. Calls `runScript(stack, scriptName, args)`, which resolves the absolute path to the script under `stacks/<stack>/scripts/` and delegates execution via `execSync`.

To add a new stack, create its directory under `stacks/` and add a registration entry to `STACK_REGISTRY` in `bin/cli.js`. No other file requires modification for routing.

---

## How `init_project.js` Works

The init script executes a **linear, ordered pipeline** of 13 async steps via `runStep()`. Each step is named, logged with `[STEP]`, and will `process.exit(1)` on failure.

| Step | Function | Description |
|------|----------|-------------|
| 1 | `assertPreconditions` | Verifies `pnpm` is available in PATH |
| 2 | `assertRequiredTemplateFiles` | Validates all template files exist before any writes |
| 3 | `createWorkspaceRoot` | Creates directories, writes `package.json`, `turbo.json`, `pnpm-workspace.yaml` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pasqualinigui/fluksos](https://github.com/pasqualinigui/fluksos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
