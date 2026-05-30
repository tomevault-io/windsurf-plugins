---
trigger: always_on
description: Lumenize is a collection of liberally licensed (MIT) and more restrictively licensed (BUSL-1.1 — Business Source License) open-source packages targeting Cloudflare's Durable Objects, which are part of Cloudflare's Workers edge computing platform. There are two complementary but distinct goals:
---

# Lumenize Project Context

## Overview
Lumenize is a collection of liberally licensed (MIT) and more restrictively licensed (BUSL-1.1 — Business Source License) open-source packages targeting Cloudflare's Durable Objects, which are part of Cloudflare's Workers edge computing platform. There are two complementary but distinct goals:
1. Provide a de✨light✨ful suite of packages that any developer can use to build scalable, high-quality, and maintainable products (MIT licensed).
2. Build the ultimate framework for vibe coding enterprise or B2B SaaS software products in a rapid and secure manner. It will be BUSL-1.1 licensed, available to enterprises via commercial licenses, and offered as a platform as a service (PaaS) with generous free tier. Nebula-related packages currently marked `UNLICENSED` pending external launch.

## Guiding Principles
- **Quality**: 
  - Code quality achieved via high test coverage: Branch >80%, Statement >90%
  - Documentation quality achieved via custom Docusaurus tooling that ensures examples always work (see Documentation section)
- **Opinionated where it matters. Flexible where it counts**: For example, the Lumenize Mesh base classes are minimal but opinionated about best practices while also providing a flexible plugin system to extend functionality along with batteries-included plugins for common use cases.
- **No foot-guns**: Vibe coders are experts in their field, but not necessarily coding or operations. Lumenize makes it easy for both the product creator AND the LLM they are using to follow best practices. For example, Durable Objects were designed to make parallel programming safer if you follow certain patterns, but will happily allow you to violate those patterns without warning. Even when Lumenize allows you to break the rules, you are loudly warned of the risks.
- **Security**: Authentication and access control are built-in and on by default. You have to jump through hoops to avoid them. At the same time, they are flexible and can be adapted to any context.

## Development Workflow Instructions

We use task files in the `tasks/` directory to track work:
- **`tasks/backlog.md`** - Small tasks and ideas for casual coding time
- **`tasks/[project-name].md`** - Active multi-phase projects with detailed plans
- **`tasks/decisions/`** - Research findings and technical decisions
- **`tasks/archive/`** - Completed projects for reference

When starting a new project, create a task file with phases and steps. See `tasks/README.md` for template and usage.

### General Development Rules
- When we change our minds on the plan from learning of earlier steps, propose updates to the task file.
- Provide clear summaries of what was implemented after each step.
- Explain design decisions and trade-offs.
- After each step/phase, ask for code review before proceeding. Ask "Ready to proceed with [next step/phase]?" after completing each step or phase.
- API changes: Mark one test as `.only` to verify the new pattern works, then update remaining tests.
- **CRITICAL SECURITY: NEVER put secrets, tokens, API keys, or credentials directly in source code files (including wrangler.jsonc, tsconfig.json, etc.). Always use `.dev.vars` files (which are gitignored) or environment variables. Tokens in wrangler.jsonc `vars` section will be committed to git.**

## How we do things around here

### Environment Variables and Secrets
**Centralized `.dev.vars` management**:
- Single root `/lumenize/.dev.vars` file (gitignored) contains all secrets
- Test directories use symlinks to the root `.dev.vars`
- `/lumenize/.dev.vars.example` provides template for contributors
- `scripts/setup-symlinks.sh` automatically creates/verifies symlinks (runs via `postinstall` hook)
- Run manually anytime: `./scripts/setup-symlinks.sh`

**When adding new test environments**:
1. Add symlink to `scripts/setup-symlinks.sh` SYMLINKS array
2. Add any new variables to root `.dev.vars` and `.dev.vars.example`
3. Run `./scripts/setup-symlinks.sh` to verify setup

### Tools
- Use `npm`. Never `pnpm` or `yarn`.
- If the library is installed never use `npx` because it requires me to approve it.

### Coding style
- Never use Typescript keyword `private`. Rather use JavaScript equivalent of starting the identifier with "#".
- **Always use synchronous storage operations** (`ctx.storage.kv.*` and `ctx.storage.sql.*`) instead of the legacy async API (`ctx.storage.get/put/delete`). This is Cloudflare's recommended pattern going forward and requires `compatibility_date: "2025-09-12"` or later.
- Storage operations are synchronous because SQLite is embedded - no async needed, no performance penalty.

### Rule of Wire Separation for Types:
- Use TypeScript types for transient in-memory constructs.
- Use TypeBox schemas for any structure that can cross a process, network, or persistence boundary.

### No build during development

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lumenize/lumenize](https://github.com/lumenize/lumenize) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-30 -->
