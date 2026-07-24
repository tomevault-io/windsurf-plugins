---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---


# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Index Network is a private, intent-driven discovery protocol built on autonomous agents. Users define "intents" and competing Broker Agents work to fulfill them through relevant connections. The system leverages LangChain/LangGraph for agent orchestration, PostgreSQL with pgvector for semantic search, and a monorepo structure with user-facing apps (`apps/web`, `apps/mac`), deployable services (`services/api`), and shared packages.

## Development Commands

### API Service

```bash
cd services/api

# Development
bun run dev                                 # Start dev server with hot reload (Bun.serve, port 3001)
bun run start                               # Start production server

# Database (Drizzle ORM)
bun run db:generate                         # Generate migrations after schema changes
bun run db:migrate                          # Apply pending migrations
bun run db:studio                           # Open Drizzle Studio (interactive DB GUI)
bun run db:seed                             # Seed database with sample data
bun run db:flush                            # Flush all data from database

# Testing
bun test                                    # Run tests with bun test
bun test tests/e2e.spec.ts                  # Run specific test file
bun test --watch                            # Run tests in watch mode

# Code quality
bun run lint                                # Run ESLint

# Maintenance/CLI tools
bun run maintenance:backfill-premises       # Backfill: enqueue enrichment for users in a network
bun run maintenance:backfill-context-hyde   # Backfill: generate HyDE docs for user contexts
bun run maintenance:backfill-global-user-contexts # Backfill: generate the global user_context (networkId=null) for every user, synthesized from active premises
bun run maintenance:backfill-intent-questions # Backfill: enqueue intent-refinement question generation (most recent active intent per user)

# Background workers
bun run integration-worker                  # Start integration sync worker
bun run social-worker                       # Start social media sync worker
```

### Web App

```bash
cd apps/web
bun run dev                                 # Start Vite dev server (with API proxy to protocol)
bun run build                               # Build blog assets then run Vite production build
bun run start                               # Start Vite preview server
bun run lint                                # Run ESLint
```

### Mac App

```bash
cd apps/mac/HaloApp
./build.sh                                  # Assemble HTML and build the macOS WKWebView app

cd ../HaloApp-iOS
./build.sh assemble                         # Regenerate mobile Resources/index.html without Xcode
./preview/build-preview.sh                  # Build the macOS preview shell for the mobile UI
```

### CLI

```bash
cd packages/cli
bun src/main.ts conversation                # Run CLI directly with Bun (no build)
bun run build                               # Build native binaries for all platforms
bun test                                    # Run CLI tests
```

> **Subtree:** `packages/cli/` mirrors `indexnetwork/cli`. Edit via this monorepo; see `### Subtrees` for sync commands.

### @indexnetwork/protocol Package

```bash
cd packages/protocol

bun run build                               # Compile TypeScript to dist/
bun run dev                                 # Watch mode
npm publish --access public                 # Publish (requires NPM login + OTP, or use CI)

# Publishing via CI (preferred):
# push dev to publish an rc prerelease
git push <indexnetwork-remote> dev

# push main to publish the stable release if the version is new
git push <indexnetwork-remote> main
```

> **Subtree:** `packages/protocol/` mirrors `indexnetwork/protocol`. Edit via this monorepo; see `### Subtrees` for sync commands.

### Subtrees

The following paths are git subtrees tracked to external repos. **Syncing is automatic for Index-owned subtrees** — the `.github/workflows/sync-subtrees.yml` workflow runs on every push to `dev` or `main` of the canonical `indexnetwork/index` repo (including PR merges), splitting each prefix and force-pushing to the corresponding subtree repo with the `SUBTREE_SYNC_PAT` secret. Subtree branches stay aligned with the monorepo branch (`dev` -> `dev`, `main` -> `main`). AgentVillage is Edge-City-owned and is mounted as a git submodule at `packages/edge-city/agentvillage`; `Edge-City/agentvillage` is canonical. The local `scripts/hooks/pre-push` hook still regenerates SKILL.md files before push, but no longer runs subtree push.

#### packages/protocol/ → indexnetwork/protocol

The `@indexnetwork/protocol` npm package (agent graphs, interfaces, tools). Two-way: edit here or in the external repo.

```bash
# Manual push if the hook failed (use dev or main)
git subtree push --prefix=packages/protocol https://github.com/indexnetwork/protocol.git <branch>

# Pull if external repo was edited directly
git subtree pull --squash --prefix=packages/protocol https://github.com/indexnetwork/protocol.git <branch>
```

#### packages/cli/ → indexnetwork/cli


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [indexnetwork/index](https://github.com/indexnetwork/index) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
