---
trigger: always_on
description: This document contains instructions for AI agents working on this codebase.
---

# Firmowid agent's instructions

This document contains instructions for AI agents working on this codebase.

This is an Elixir LiveView app. It is focused on invoicing, bank accounts synchronization,
and other tasks related to company management, like payroll or client's billing.

## Development server

When running, assume that the whole application - with all required services -
is properly running. We use `wt` (`worktrunk`) to maintain each worktree.
Parallel worktrees are running on the same host, so we use different ports.
They are stored in `.env.worktree` and `.server.port` files and are used by
development scripts.

**Accessing the dev server:**

- Check `.server.port` for the port number - for example: `http://localhost:16423`
- Use `kira@bytecraft.collective` / `kolejka123456` as credentials (look at
  `seeds.exs` if in doubt)
- Visit `/development` for the full seed data guide

**Local setup:**

- `mix dev.up` - starts services, reads config from `.env.worktree` (or uses defaults)
- `mix dev.down` - stops services
- Worktrunk generates `.env.worktree` with hashed ports for feature branches

Tidewave MCP should be available, allowing you to inspect the running system.
If not - flag that instantly. It's the best way to debug so if it's missing
**it's a huge issue.**

Do not start own servers or restart. Ask the user to do that if something is
malfunctioning. You can by mistake kill other worktrees / your MCPs / your
process running on this machine.

## Use git

Before starting the work, ask user if they want you to commit the changes.
If they say so, after each successful, atomic change - commit it.
Make sure it works and is correct before doing so. Instructions below.

All commit messages must follow **Conventional Commits (Angular variant)**:

- required format: `<type>(<scope>): <subject>`
- `scope` is optional
- do not use merge commits (`Merge branch ...`) - they are blocked in CI

Examples:

- `feat(payroll): add monthly summary action`
- `fix: prevent duplicate invoice import`
- `chore(ci): enforce conventional commits in workflow`

## Manual testing and good will

While we employ a bunch of tools to analyze code and catch bugs early, it's
still important to test your changes manually. This is especially true for
changes that affect the user interface or are introducing something new.

Always go above and beyond to make sure that that whoever comes after you
understands what you've done, that it works and that it's correct.

## Automated Tests

They have value, and we have these in our codebase, as part of our validation suite.
They have to be super fast, test critical paths for regressions.

Do not **overtest**. Discuss with users about testing; we never want to be held back
by outdated / inflexible tests.


## Migrations Policy (Ash only)

Schema migrations must be managed through Ash/AshPostgres migration tooling.

- Use Ash migration/codegen flow (`mix ash.codegen`, `mix ash.migrate`, and
  underlying `mix ash_postgres.generate_migrations`).
- Do **not** generate direct Ecto migrations (`mix ecto.gen.migration`) in normal work.
- Direct/manual Ecto migrations are allowed only in emergency incidents when Ash
  tooling cannot express the change in time.
- Every emergency manual migration must be explicitly documented in PR/commit notes,
  with reason and follow-up plan to return to Ash-managed schema changes.

## Code Quality Verification

Before submitting changes, always run the quality checks:

```bash
mix check
```

> Do not `tail` or `grep` the output - it's super compact, specifically for agents.

This single command runs all quality checks in order:

1. `mix compile --warnings-as-errors` - Compile with strict warnings
2. `mix format --check-formatted` - Verify code formatting  
3. `mix credo --strict` - Static code analysis
4. `mix sobelow --config` - Security vulnerability scanning
5. `mix test` - Run test suite

All checks must pass before changes can be merged.

## Code Style Guidelines

### Frontend architecture conventions (`lib/firmowid_web`)

- **Directory path = module namespace** (`FirmowidWeb.X.Y.Z` lives in
  `lib/firmowid_web/x/y/z.ex`).
- Prefer a **feature-first layout** with standard subfolders:
  `views/`, `components/`, `controllers/`, `utilities/`.
- Keep dependencies directional: nested/child features may depend on shared
  parent feature modules, but avoid cross-sibling dependencies.
- Keep these conventions enforceable in practice (Credo + code review), and
  always finish refactors with `mix check`.

### User-facing URL language

- User-facing pathnames, query param keys, and human-readable query values must be
  **Polish**.
- Technical/internal endpoints may stay technical when they are not part of the
  product UX (for example auth internals, admin routes, health checks, or external
  callback payloads).
- Keep canonical feature URLs in feature-owned navigation modules (for example
  `FirmowidWeb.Invoicing.Navigation`).
- Put shared parsing/encoding mechanics in `FirmowidWeb.Infrastructure.Utilities.*`.
- Do not add compatibility aliases or English fallbacks unless the user explicitly
  asks for a staged migration.

### Pure Elixir Files Preference

Avoid writing `.heex` templates. Sometimes, it's unavoidable, but prefer Elixir

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Alergeek-Ventures/firmowid](https://github.com/Alergeek-Ventures/firmowid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
