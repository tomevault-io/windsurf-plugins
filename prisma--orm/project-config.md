---
trigger: always_on
description: Welcome. This is a contract‑first, agent‑friendly data layer.
---

# Agents — Prisma Next

Welcome. This is a contract‑first, agent‑friendly data layer.

## Start Here

- [Docs Index](docs/README.md) — How the docs are organized and what to read next
- [Architecture Overview](docs/Architecture%20Overview.md) — High-level design principles
- [Testing Guide](docs/Testing%20Guide.md) — Philosophy, patterns, and commands
- [Rules Index](.agents/rules/README.md) — All agent rules organized by topic
- [ADRs](docs/architecture%20docs/adrs/) — Architecture Decision Records

### Modular Onboarding

- [Getting Started](docs/onboarding/Getting-Started.md) — Build, test, and run demo
- [Repo Map & Layering](docs/onboarding/Repo-Map-and-Layering.md) — Package organization and import rules
- [Conventions](docs/onboarding/Conventions.md) — TypeScript, tooling, and code style
- [Testing](docs/onboarding/Testing.md) — Test commands, patterns, and organization
- [Common Tasks Playbook](docs/onboarding/Common-Tasks-Playbook.md) — Add operation, split monolith, fix import
- [Cursor Cloud Agents](docs/onboarding/Cursor-Cloud-Agents.md) — Setup, secrets, lockfile discipline, snapshot management

## Project Overview

**Prisma Next** is a contract-first data access layer:

- **Contract-first**: Emit `contract.json` + `contract.d.ts` — no executable runtime code generation
- **Composable DSL**: Type-safe query builder (`sql().from(...).select(...)`)
- **Machine-readable**: Structured artifacts that agents can understand and manipulate
- **Runtime verification**: Contract hashes and guardrails ensure safety before execution

## Where skills and rules live

The repo keeps a single canonical home for each kind of agent surface, with presentation symlinks that satisfy the paths each harness expects.

- **Skills — canonical home:** `skills-contrib/<skill-name>/SKILL.md`. These are the tracked, deliverable source-of-truth files.
- **Skills — presentation symlinks:** `.claude/skills/<skill-name>` and `.agents/skills/<skill-name>` are symlink directories pointing into `skills-contrib/`. They exist so the various agent harnesses (Cursor, Claude Code, …) can find the skills at the paths they expect. Both symlink trees are gitignored.
- **Skills — wired post-install:** the symlink trees (and any tooling-specific copies) are materialized by the `prepare` script in [`package.json`](./package.json), which runs `skills add ./skills-contrib --skill '*' --agent universal claude-code -y`. After `pnpm install`, `.claude/skills/` and `.agents/skills/` are populated automatically — no manual setup. If you add a new skill under `skills-contrib/`, re-run `pnpm install` (or just the prepare hook) to wire it into the tooling locations. `pnpm lint:skills` validates skill frontmatter in CI.
- **Drive upstream in ignite:** `drive-*` skills and [`docs/drive/`](https://github.com/prisma/ignite/tree/main/docs/drive) live in [prisma/ignite](https://github.com/prisma/ignite) (`skills/.pilot/`); install with `npx skills add prisma/ignite/skills/.pilot --skill '*'`. This repo keeps only `drive/` project-context overlays.
- **Rules — canonical home:** `.agents/rules/<rule-name>.mdc` — the only tracked copy (rule files must use the `.mdc` extension; a `.md` rule is never loaded by the harnesses) (a whitelist exception in `.gitignore` keeps `.agents/rules/**` tracked even though the rest of `.agents/` is ignored). The `.cursor/rules/` and `.claude/rules/` trees are gitignored presentation mirrors containing only relative symlinks back into `.agents/rules/`.
- **Rules — wired post-install:** like skills, the symlink trees are materialized by `prepare` (which runs `node scripts/sync-agent-rules.mjs` after `skills add`). After `pnpm install` the trees are regenerated automatically — no manual setup. A rule added only to `.cursor/rules` is gitignored and silently lost, so **always add it under `.agents/rules/`** and run `pnpm rules:sync`. `pnpm lint:rules:symlinks` enforces tree/canonical consistency in CI.
- **Practical implication for editors and sub-agents:** when amending or authoring a skill or rule, **edit at the canonical path** (`skills-contrib/` for skills, `.agents/rules/` for rules) — not at the symlinked path. An edit through a symlink writes to the canonical file on disk, but `git status` and `git ls-files` report against the canonical path; addressing the canonical path keeps diffs legible and avoids surface churn.

## Golden Rules

- **Node.js version**: Use the shell's Node — do not run `nvm`/`fnm` or any version switcher. Source of truth is the root `package.json` `engines.node`. If the shell's `node -v` doesn't satisfy that, report that the shell is misconfigured (e.g. user should set their default Node in their version manager, or use Volta) — don't try to switch it yourself.
- Use `pnpm`, not `npm`. Never use `npx`.
- Build with `pnpm build` (delegates to Turbo). After changing exported types in a workspace package consumed elsewhere, run that package's `pnpm build` to refresh `dist/*.d.mts` before validating downstream TypeScript.
- For typecheck/test, use the local `pnpm typecheck` / `pnpm test` scripts rather than writing `tsc`/`vitest` invocations from scratch.
- Use arktype, not zod.
- Never add file extensions to imports in TypeScript.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prisma/orm](https://github.com/prisma/orm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
