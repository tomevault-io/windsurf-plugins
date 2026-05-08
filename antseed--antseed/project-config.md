---
trigger: always_on
description: **Never commit or push unless explicitly asked.** Default workflow:
---

# CLAUDE.md -- Agent Context for AntSeed Monorepo

## Commit & push policy (IMPORTANT)

**Never commit or push unless explicitly asked.** Default workflow:

1. Make the requested edits.
2. Run typecheck / build / tests to verify the change is sound.
3. **Stop.** Leave changes in the working tree (staged or unstaged). Report
   what you did and wait.

Only commit/push when the user explicitly says so — e.g. "commit", "push",
"/commit", "ship it", "open a PR", or invokes the `commit`, `commit-amos`,
`pr-create`, or `pr-merge` skills.

Rationale: the user batches related edits into a single reviewed commit with
their own message. Auto-committing every tweak produces a noisy history of
one-line commits that has to be squashed later. Do not chain `edit → build
→ commit → push` as a reflex — even if prior turns in the same session did.

## Project Overview
AntSeed is a peer-to-peer AI services network. Providers offer differentiated
AI services and buyers route requests to the best available peer. This monorepo
contains the core SDK, provider/router plugins, CLI, desktop app, dashboard,
and website.

**Important context for all documentation and code:** AntSeed is NOT for raw
resale of API keys or subscription access. Providers must add value (TEE, skills,
agents, fine-tuned models, managed products). Subscription-based plugins
(provider-claude-code, provider-claude-oauth) are for testing/development only.
Reselling subscription credentials violates upstream provider terms of service.

## Repository Structure
```
packages/           Core libraries (published to npm as @antseed/*)
  node/             Core protocol SDK — P2P, discovery, metering, payments
  provider-core/    Shared provider infrastructure
  router-core/      Shared router infrastructure
plugins/            Provider and router plugins (extend core)
  provider-anthropic/     Anthropic API key provider
  provider-claude-code/   Claude Code keychain provider
  provider-claude-oauth/  Claude OAuth provider
  provider-openai/        OpenAI-compatible provider (OpenAI, Together, OpenRouter)
  provider-local-llm/     Local LLM (Ollama/llama.cpp) provider
  router-local/           Local router (Claude Code, Aider, Continue.dev)
apps/               Applications
  cli/              CLI tool (@antseed/cli) — bin: antseed
  desktop/          Electron desktop app
  dashboard/        Web dashboard (Fastify server + React frontend)
  website/          Marketing website (React + Vite + Tailwind)
e2e/                End-to-end tests
docs/protocol/      Protocol specification and plugin templates
```

## Tech Stack
- **Runtime**: Node.js >=20, ES modules throughout
- **Language**: TypeScript 5.x with strict mode
- **Package Manager**: pnpm workspaces
- **Build**: tsc for libraries, Vite for web apps and Electron renderer
- **Test**: vitest for all packages
- **Native Modules**: better-sqlite3, node-datachannel (in packages/node)

## Dependency Graph
```
@antseed/node (no internal deps)
  ├── @antseed/provider-core (peer: node)
  │     └── provider-anthropic, provider-claude-code, provider-claude-oauth,
  │         provider-openai, provider-local-llm
  ├── @antseed/router-core (peer: node)
  │     └── router-local
  ├── antseed-dashboard (peer: node)
  │     └── @antseed/cli (depends: node + dashboard)
  │           └── @antseed/desktop (builds: cli + dashboard)
  └── @antseed/website (standalone, no internal deps)
```

## Key Commands
```bash
pnpm install          # Install all dependencies
pnpm run build        # Build all packages in dependency order
pnpm run test         # Run all tests
pnpm run typecheck    # Type-check all packages
pnpm run clean        # Remove all dist/ directories
```

## Build Order
Build must respect: node -> provider-core/router-core -> plugins -> dashboard -> cli -> desktop.
The root build script handles this via tiered execution.

## Internal Dependencies
All cross-package references use `workspace:*` protocol in package.json.
Peer dependencies (e.g., @antseed/node) are declared in peerDependencies
and resolved via the workspace.

## Plugin Architecture
- Provider plugins implement the `AntseedProviderPlugin` interface from @antseed/node
- Router plugins implement the `AntseedRouterPlugin` interface from @antseed/node
- Both extend shared infrastructure from provider-core / router-core
- The CLI loads plugins dynamically via the registry in apps/cli/src/plugins/registry.ts

## Smart Contracts
```
packages/contracts/
├── interfaces/              Shared Solidity interfaces (IAntseed*.sol, IERC8004Registry.sol)
├── AntseedRegistry.sol      Central address book for all protocol contracts
├── AntseedStaking.sol       Seller staking (holds stake USDC, binds to agentId)
├── AntseedDeposits.sol      Buyer deposits, seller payouts (holds buyer USDC)
├── AntseedChannels.sol      Payment channel lifecycle, ReserveAuth + SpendingAuth (swappable, holds NO USDC)
├── AntseedEmissions.sol     ANTS token emissions (USDC volume-based)
├── AntseedSlashing.sol      Swappable slashing logic for staked sellers
├── AntseedSubPool.sol       Subscription pool
├── ANTSToken.sol            ANTS ERC-20 token (52M max supply)
├── MockERC8004Registry.sol  Mock ERC-8004 IdentityRegistry (local testing only)
└── MockUSDC.sol             Test USDC
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AntSeed/antseed](https://github.com/AntSeed/antseed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
