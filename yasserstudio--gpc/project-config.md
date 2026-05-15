---
trigger: always_on
description: GPC is a TypeScript CLI for the Google Play Developer API v3. Monorepo with Turborepo + pnpm.
---

# GPC - Project Instructions

## What is this?

GPC is a TypeScript CLI for the Google Play Developer API v3. Monorepo with Turborepo + pnpm.

## Project Structure

```
packages/cli       → @gpc-cli/cli (bin: gpc) — CLI entry point, Commander.js
packages/core      → @gpc-cli/core — business logic, command orchestration
packages/api       → @gpc-cli/api — typed Google Play API client
packages/auth      → @gpc-cli/auth — service account, OAuth, ADC
packages/config    → @gpc-cli/config — config loading, env vars, profiles
packages/plugin-sdk → @gpc-cli/plugin-sdk — plugin interface
plugins/plugin-ci  → @gpc-cli/plugin-ci — CI/CD helpers
apps/docs/         → VitePress docs site (single source of truth for all documentation)
.dev/              → Private docs (marketing, strategy, engineering, competitive) — gitignored
e2e/               → End-to-end tests
```

## Key Conventions

- TypeScript strict mode, ESM-first
- Named exports only (no default exports)
- Barrel exports via index.ts per package
- File naming: kebab-case (e.g., rate-limiter.ts)
- Conventional commits: feat(scope), fix(scope), docs(scope)
- Scopes: cli, core, api, auth, config, plugin-sdk, ci, docs
- No circular dependencies between packages
- Dependency flow: cli → core → api, auth, config

## Current Status

- Phase 0 ✓ — Monorepo scaffold
- Phase 1 ✓ — Auth, config, CLI shell
- Phase 2 ✓ — API client, edits lifecycle, apps commands
- Phase 3 ✓ — Releases, tracks, rollouts, upload, promote
- Phase 4 ✓ — Listings, metadata, images, Fastlane compat
- Phase 5 ✓ — Reviews, vitals, reporting API, CI threshold alerting
- Phase 6 ✓ — Subscriptions, IAP, purchases, pricing, regional conversion
- Phase 7 ✓ — Reports, users, testers, grants, CSV import
- Phase 8 ✓ — Plugin SDK, plugin manager, lifecycle hooks, plugin-ci
- Phase 9 ✓ — Security audit, interactive mode, VitePress docs, standalone binary, Homebrew tap, npm publish, README/CHANGELOG
- Phase 10 ✓ — `gpc preflight` pre-submission compliance scanner (9 scanners, offline AAB policy checks)
- Published to npm: `npm install -g @gpc-cli/cli`
- Current version: v0.9.71 — pre-release series (`0.9.x` → `1.0.0` public launch)
- v0.9.71: `gpc doctor` quota proximity check (warns at >80% daily/per-minute API usage) and plugin health check (discovers, loads, reports each plugin).
- v0.9.70: `--in-app-update-priority <0-5>` and `--retain-version-codes <csv>` flags on upload, `default.txt` Fastlane-style changelog fallback via `--notes-dir`, promote preserves `inAppUpdatePriority` + `name`, GitHub Actions Node.js 22.
- 2,269 total tests, 7 packages building, 90%+ line coverage on all core packages
- Changelog-generation series (v0.9.61 → v0.9.64): complete. `gpc changelog generate --target play-store --locales auto --ai --apply` does commit → translated Play Store notes → written into draft release, one command.
- v0.9.63 highlight: AI-assisted Play Store translation. `gpc changelog generate --target play-store --locales auto --ai` translates non-source locales via the user's own LLM key. Auto-detects env priority `AI_GATEWAY_API_KEY` → `ANTHROPIC_API_KEY` → `OPENAI_API_KEY` → `GOOGLE_GENERATIVE_AI_API_KEY`. Non-reasoning model defaults. Gateway path unlocks cost-per-run in USD. All 4 AI SDK deps lazy-loaded (cold-start preserved).
- v0.9.62 highlight: multilingual Play Store release notes. `gpc changelog generate --target play-store --locales <csv|auto>` emits per-locale "What's new" text with 500-char budget enforcement.
- v0.9.61 highlight: smarter changelog generation. `gpc changelog generate` clusters git commits via Union-Find on file-path overlap + Jaccard keyword similarity + time proximity, lints subjects against project voice, emits canonical GitHub Release markdown / JSON / LLM prompt.
- v0.9.56 highlight: first Android publishing CLI with Managed Google Play support (`gpc enterprise publish` via Play Custom App Publishing API).
- GitHub Releases: umbrella `v*` tags only, user-facing notes (see `apps/docs/advanced/conventions.md` for template)

## Testing

- Vitest for all tests (2,269 total across 7 packages + e2e)
- Tests in `tests/` directory per package
- Mock external APIs — never call real Google APIs in tests
- Mock fetch with `vi.stubGlobal("fetch", mockFn)` for API tests
- Run: `pnpm test` or `pnpm test --filter @gpc-cli/<package>`

## Building

- tsup for bundling each package
- Turborepo for task orchestration
- Run: `pnpm build` (builds all with dependency order)

## Documentation

VitePress docs site: https://yasserstudio.github.io/gpc/
Source: `apps/docs/` — single source of truth for all documentation (guide, commands, CI/CD, advanced, migration, reference)

Key pages for contributors:

- `apps/docs/advanced/architecture.md` — system design, package graph
- `apps/docs/advanced/conventions.md` — code style, git, testing
- `apps/docs/advanced/security.md` — credential handling, threat model
- `apps/docs/guide/developer-verification.md` — Google's 2026 verification rollout and GPC support plan

Private docs in `.dev/` (gitignored) — internal use only.

## GPC Skills

Agent skills for GPC workflows are in `.agents/skills/gpc-*`.

Install: `gpc install-skills` (interactive wizard) or `npx skills add yasserstudio/gpc-skills`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yasserstudio/gpc](https://github.com/yasserstudio/gpc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
