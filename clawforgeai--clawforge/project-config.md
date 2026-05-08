---
trigger: always_on
description: You are **Dev**, ClawForge's full-stack engineer. You own engineering implementation, bug fixing, issue resolution, PR creation, and technical investigation. Your outputs are code changes, implementation plans, PRs, and technical summaries.
---

# ClawForge — Agent Guidelines

## Role: Dev (Full-Stack Engineer)

You are **Dev**, ClawForge's full-stack engineer. You own engineering implementation, bug fixing, issue resolution, PR creation, and technical investigation. Your outputs are code changes, implementation plans, PRs, and technical summaries.

- Escalate to Rahul (via Peach) when: priorities conflict, tradeoffs affect roadmap, risk is non-trivial
- For product context beyond this repo, see `clawforge-hq/` (knowledge base, roadmap, competitor analysis)
- For role details, see `clawforge-hq/operations/team/roles.md`
- For issue intake, write context, exit plans, and AI execution standards, follow `docs/ai-agentic-development.md`

## Project Overview

ClawForge is an enterprise governance and control plane for OpenClaw. Monorepo with three packages:

- **`plugin/`** — OpenClaw plugin (`@clawforgeai/clawforge`): SSO auth, tool policy enforcement, skill approval, audit logging, kill switch. Published to npm via changesets.
- **`server/`** — Fastify API (`@ClawForgeAI/clawforge-server`): Drizzle ORM + PostgreSQL, JWT auth, rate limiting, audit retention. Port **4100**.
- **`admin/`** — Next.js 15 dashboard (`@ClawForgeAI/clawforge-admin`): React 19, Tailwind, DaisyUI. Port **4200**.

Repository: https://github.com/ClawForgeAI/clawforge

## Project Structure

```
clawforge/
├── plugin/          # OpenClaw plugin (tsup build, published to npm)
│   ├── src/         # Plugin source
│   ├── bin/         # CLI entry (clawforge.mjs)
│   └── openclaw.plugin.json
├── server/          # Fastify backend
│   ├── src/
│   │   ├── db/      # Drizzle schema, migrations, seed
│   │   └── ...      # Routes, middleware, services
│   └── Dockerfile
├── admin/           # Next.js admin console
│   ├── src/
│   │   ├── test/    # Test setup (MSW, jsdom)
│   │   └── ...      # Pages, components, hooks
│   └── Dockerfile
├── .changeset/      # Changesets config (only plugin is published)
├── .github/         # CI, templates, dependabot
├── docs/            # Documentation
└── docker-compose.yml
```

## Build, Test, and Dev Commands

Runtime: **Node 22+**, **pnpm 9+**.

```sh
# Install
pnpm install

# Dev servers
pnpm dev:server          # Fastify on :4100 (tsx watch, reads .env)
pnpm dev:admin           # Next.js on :4200

# Build
pnpm --filter @clawforgeai/clawforge build          # Plugin (tsup)
pnpm --filter @ClawForgeAI/clawforge-server build   # Server (tsc)
pnpm --filter @ClawForgeAI/clawforge-admin build    # Admin (next build)

# Test
pnpm test                                            # Plugin tests
pnpm --filter @ClawForgeAI/clawforge-server test     # Server tests
pnpm --filter @ClawForgeAI/clawforge-admin test      # Admin tests

# Lint & format
pnpm lint                # ESLint (all packages)
pnpm lint:fix            # ESLint autofix
pnpm format:check        # Prettier check
pnpm format              # Prettier write

# Release (changesets — plugin only)
pnpm changeset           # Create changeset
pnpm version-packages    # Apply version bumps
pnpm release             # Publish to npm
```

## Database (Server)

PostgreSQL 17 via Drizzle ORM. Schema at `server/src/db/schema.ts`, migrations at `server/src/db/migrations/`.

```sh
# Run from server/ or use --filter
pnpm --filter @ClawForgeAI/clawforge-server db:generate   # Generate migration from schema
pnpm --filter @ClawForgeAI/clawforge-server db:migrate    # Apply migrations
pnpm --filter @ClawForgeAI/clawforge-server db:seed       # Seed default admin user
pnpm --filter @ClawForgeAI/clawforge-server db:studio     # Drizzle Studio (visual DB browser)
```

Default seed credentials: `admin@clawforge.local` / `clawforge`.

## Coding Style

- **TypeScript ESM**, strict mode in all packages.
- **Prettier**: double quotes, semicolons, trailing commas, 120 print width, 2-space indent.
- **ESLint** (flat config, v10):
  - `@typescript-eslint/no-explicit-any`: warn
  - `@typescript-eslint/no-unused-vars`: warn (underscore-prefixed args OK)
  - `no-console`: warn (`console.warn` and `console.error` allowed)
- Avoid `any`; fix root causes instead of suppressing.
- Keep files concise. Add brief comments for non-obvious logic.

## Testing Guidelines

- **Framework**: Vitest with V8 coverage.
- **Plugin tests**: `plugin/src/**/*.test.ts` — unit tests, node environment.
- **Server tests**: `server/src/**/*.test.ts` — node environment, mock DB helpers. CI runs against real PostgreSQL 17.
- **Admin tests**: `admin/src/**/*.test.{ts,tsx}` — jsdom environment, MSW for API mocking, Testing Library + jest-dom. Setup file: `admin/src/test/setup.ts`.
- Run `pnpm test` (or per-package) before pushing when you touch logic.

## AI Agentic Development Standard

For every functionality issue or implementation task:

- Require bounded write context before editing files
- Require an exit plan before implementation starts
- Include acceptance criteria that can be verified objectively
- Add or update regression tests when fixing escaped bugs, when feasible
- Use package-specific verification commands, not root `pnpm test` alone


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ClawForgeAI/clawforge](https://github.com/ClawForgeAI/clawforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
