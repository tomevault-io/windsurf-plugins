---
trigger: always_on
description: This file provides guidance for AI coding agents (Claude Code, Cursor, Codex, Amp, etc.) when working with code in this repository.
---

# AGENTS.md - Comprehensive Guide for AI Coding Agents

This file provides guidance for AI coding agents (Claude Code, Cursor, Codex, Amp, etc.) when working with code in this repository.

## Essential Commands - Quick Reference

### Build & Development
- **Build**: `pnpm build` (root) or `turbo build`
- **Dev**: `pnpm dev` (root) or navigate to package and run `pnpm dev`
- **Setup (core)**: `pnpm setup-dev` — core DBs (Doltgres, Postgres, SpiceDB), env config, migrations, admin user
- **Setup (isolated)**: `pnpm setup-dev --isolated <name>` — same as above but in a parallel environment (see [Isolated Environments](#isolated-parallel-environments))
- **Setup (optional services)**: `pnpm setup-dev:optional` — Nango + SigNoz + OTEL + Jaeger (run `setup-dev` first)
- **Optional services lifecycle**: `pnpm optional:stop` | `pnpm optional:status` | `pnpm optional:reset`

### Verification

**Pre-push** (run both, in order):
```bash
pnpm format     # auto-fix formatting
pnpm check      # lint + typecheck + test + format:check + env-descriptions + route-handler-patterns + dal-boundary + knip
```

**Single-command iteration:** `pnpm typecheck`, `pnpm lint` (`lint:fix`), `pnpm test`, `cd <pkg> && pnpm test --run <file>`

### Database Operations (run from monorepo root)
- **Generate migrations**: `pnpm db:generate` - Generate Drizzle migrations from schema changes
- **Apply migrations**: `pnpm db:migrate` - Apply generated migrations to database
- **Drop migrations**: `pnpm db:drop` - Drop migration files (use this to remove migrations, don't manually delete)
- **Database studio**: `pnpm db:studio` - Open Drizzle Studio for database inspection
- **Check schema**: `pnpm db:check`
- **Initialize auth**: `pnpm db:auth:init` - Create default organization and admin user for local development

### Creating Changelog Entries (Changesets)

Create a changeset for any user-facing change to a published package:

```bash
pnpm bump <patch|minor|major> --pkg <package> "<message>"
```

**Examples:**
```bash
# Single package
pnpm bump patch --pkg agents-core "Fix race condition in agent message queue"

# Multiple packages (for tightly coupled changes)
pnpm bump minor --pkg agents-sdk --pkg agents-core "Add streaming response support"
```

**Valid package names:** `agents-cli`, `agents-core`, `agents-api`, `agents-manage-ui`, `agents-work-apps`, `agents-sdk`, `create-agents`, `ai-sdk-provider`

**Semver guidance:**
- **Major**: Reserved - do not use without explicit approval
- **Minor**: Schema changes requiring migration, significant behavior changes
- **Patch**: Bug fixes, additive features, non-breaking changes

#### Writing Good Changelog Messages

**Target audience:** Developers consuming these packages

**Style requirements:**
- Sentence case: "Add new feature" not "add new feature"
- Start with action verb: Add, Fix, Update, Remove, Improve, Deprecate
- Be specific about what changed and why it matters to consumers
- Keep to 1-2 sentences

**Good examples:**
- "Fix race condition when multiple agents connect simultaneously"
- "Add `timeout` option to `createAgent()` for custom connection timeouts"
- "Remove deprecated `legacyMode` option (use `mode: 'standard'` instead)"
- "Improve error messages when agent registration fails"

**Bad examples:**
- "fix bug" (too vague - which bug? what was the impact?)
- "update dependencies" (not user-facing, doesn't need changeset)
- "Refactored the agent connection handler to use async/await" (implementation detail, not user impact)
- "changes" (meaningless)

**When to create a changeset (MANDATORY):**
- Any bug fix, feature, or behavior change to a published package — even if the package is "internal-facing" (e.g., `agents-work-apps`, `agents-api`). If the code ships to users or affects runtime behavior, it needs a changeset.
- This includes work-app integrations (Slack, GitHub), API route changes, SDK changes, CLI changes, and core library changes.

**When NOT to create a changeset:**
- Documentation-only changes
- Test-only changes
- Internal tooling/scripts changes
- Changes to ignored packages (agents-ui, agents-docs, cookbook-templates, test-agents)

**Multiple changes in one PR:**
If a PR affects multiple packages independently, create separate changesets for each with specific messages. If changes are tightly coupled (e.g., updating types in core that SDK depends on), use a single changeset listing both packages.

### Running Examples / Reference Implementations
- Use `agents-cookbook/` for reference implementations and patterns.
- There is no `examples/` directory; prefer cookbook recipes or package-specific README files.

### Documentation Development
```bash
# From agents-docs directory
pnpm dev              # Start documentation site (port 3000)
pnpm build           # Build documentation for production
```

## Code Style (Biome enforced)
- **Imports**: Use type imports (`import type { Foo } from './bar'`), organize imports enabled, barrel exports (`export * from './module'`)
- **Formatting**: Single quotes, semicolons required, 100 char line width, 2 space indent, ES5 trailing commas
- **Types**: Explicit types preferred, avoid `any` where possible (warning), use Zod for validation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [inkeep/agents](https://github.com/inkeep/agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
