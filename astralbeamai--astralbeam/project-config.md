---
trigger: always_on
description: - Use the product glossary consistently: an Organization is an AstralBeam customer (typically a SaaS app), organization users are that customer's employees who use the AstralBeam dashboard, Tenants are the Organization's customers, and tenant users (`TenantUser`) are the Tenants' users who interact with the embedded agent sidebar.
---

# AstralBeam development

- Use the product glossary consistently: an Organization is an AstralBeam customer (typically a SaaS app), organization users are that customer's employees who use the AstralBeam dashboard, Tenants are the Organization's customers, and tenant users (`TenantUser`) are the Tenants' users who interact with the embedded agent sidebar.
- Organizations have immutable UUIDs and editable slugs. Only the organization table may have a `slug` column. Use organization slugs only for URLs and their validation, never as identity in credentials, tokens, relationships, or seed lookups. Changing the slug breaks old URLs.
- Other entities use opaque UUIDs. First-party organization-owned tables use `(organization_id, id)` primary keys. Better Auth tables retain their adapter-compatible keys. Compose public agent IDs as `agent_<organizationId>_<id>` and API-key IDs as `key_<organizationId>_<id>`.

## Tooling and validation

- Use Deno from the affected project directory (`webapp`, `www`, `sdk`, or `examples/todos`) with `deno task <script>`, or from the repository root with `deno task --cwd <project> <script>`. The projects do not form a package-manager workspace. Deno is the only supported repository JavaScript runtime and package manager. Vite and npm tooling run through its compatibility layer. SDK consumer examples may use the host application's package manager.
- Keep every project's `check`, `test`, and `build` tasks meaning the same thing, and `ready` meaning `check`, `test`, and `build`. `www` alone runs `build` before `test`, because its test reads `dist/`.
- Compose reusable validation gates in the affected project's `check` task so local and CI checks stay aligned. Keep workflow additions limited to environment provisioning and checks that require a distinct execution environment.
- Root `tsconfig.base.json` holds only the compiler options all four projects share. Each `tsconfig.json` `extends` it and keeps its own `jsx`, `lib`, `types`, `paths`, and file globs.
- Keep the root `deno.jsonc` a launcher for the four projects, with `format` and `format:check` tasks and formatting configuration limited to files directly in the root. CI must run the root `format:check` task. Do not add a `workspace` field, project-specific lint configuration, or root copies of per-project tasks such as `check`, `test`, and `ready`.
- Before non-trivial changes, inspect the current code, instructions, Git base and diff, generated artifacts, and installed APIs. Prefer supported upstream contracts, narrow diffs that preserve original names and code structure where semantics allow, and removing one-use helpers over custom plumbing or speculative abstraction.
- Ask before writing or changing code outside the user's explicitly requested scope. Research, planning, and instruction updates do not authorize implementation or resuming previously paused implementation.
- Before running `deno task knip:fix`, commit or back up untracked work because it can delete unused files that Git cannot restore. Then inspect the complete project diff before running `deno task check:fix`.
- Reserve Knip entries for actual execution or externally discovered roots, and add reusable modules only when code uses them. Do not hide speculative modules or accidental exports with entries. Keep `includeEntryExports` enabled in the three applications, whose entry exports must also be used in-project, and off in `sdk`, whose entry-point exports are the published npm surface.
- Run `scripts/setup.sh` once after pulling to install the OS-level tooling and the projects' frozen dependencies. Otherwise, use the smallest relevant project task or syntax/configuration check. Documentation and instruction changes need only source review and `git diff --check`.
- Do not automatically run `deno task check`, `deno task test`, or `deno task ready`. `ready` already runs checks, tests, and builds. Run it once before creating a PR or when explicitly requested, without separate `check` or `test` runs unless diagnosing a failure.
- Run JavaScript and TypeScript tests with Vitest through the project's Deno task, never `Deno.test` or `deno test`. The `webapp` and todos browser suites use Playwright through their own `e2e` tasks, which stay out of `check`, `test`, `ready`, and CI.
- When coding or reviewing, remove low-value or redundant tests and unnecessary fixtures/mocks. Keep tests that protect durable behavior, security boundaries, or observed regressions. Prefer short, direct tests and code over verbose setup or abstractions unless the extra complexity catches a distinct, worthwhile failure. Do not retain tests merely to restate implementation details, trivial constants, or generated structure.
- Before final validation, turn durable, non-obvious user corrections into one concise, nonduplicative instruction in the closest `AGENTS.md` or skill. Skip one-off decisions and preferences.

## Documentation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AstralBeamAI/astralbeam](https://github.com/AstralBeamAI/astralbeam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
