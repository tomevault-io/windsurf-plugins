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

## Changelog policy for PRs

When opening a PR, update `CHANGELOG.md` for any user-facing package, CLI,
desktop, protocol, configuration, or documentation behavior change. If a PR has
no user-facing change, say that explicitly in the PR description.

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

**AntSeed is not a crypto company.** Settlement happens in USDC on Base, but
buyers are not required to already hold or handle crypto: the desktop app
offers a card checkout (Stripe-backed onramp, region-gated) alongside crypto
funding, and USDC can be funded from an exchange withdrawal, another wallet,
or a card on-ramp (see `apps/website/docs/guides/payments.md`,
`apps/desktop/src/renderer/ui/components/views/VprDepositView.tsx`, and
`apps/cli/README.md`). Never describe AntSeed as crypto-only, wallet-only, or
as requiring the user to already own cryptocurrency.

**When writing marketing or blog content that states a fact about AntSeed**
(pricing, payment flow, CLI behavior, protocol mechanics, what is or isn't
free) — source it only from this repository (code, docs, CHANGELOG) or the
live antseed.com website. Never state an AntSeed-specific fact from general
knowledge, a web search, or inference from silence in a doc (e.g. "the doc
doesn't mention a free tier, so there must not be one" is not a valid basis
for a claim). If a fact can't be verified from those two sources, say so
explicitly rather than asserting it.

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
  ants/             ANTS staking dashboard (@antseed/ants: Fastify server + React web + service layer shared with the CLI)
  website/          Marketing website (React + Vite + Tailwind)
e2e/                End-to-end tests
docs/protocol/      Protocol specification and plugin templates
```

## Tech Stack
- **Development runtime**: Node.js 24 (pinned in `.nvmrc` and Volta), ES modules throughout. Published package engine ranges and Electron's embedded runtime are separate from this development pin.
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

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Antseed/antseed](https://github.com/Antseed/antseed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
