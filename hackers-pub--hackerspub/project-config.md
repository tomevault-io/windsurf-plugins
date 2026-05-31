---
trigger: always_on
description: Guide for LLM-Powered Agents
---

Guide for LLM-Powered Agents
============================

This file provides guidance to LLM-powered agents when working with code in
this repository.


AI Policy Compliance
--------------------

> [!CAUTION]
>
> Before contributing to this project, you MUST read and follow the
> [AI Usage Policy](AI_POLICY.md).
>
> All AI usage must be disclosed in pull requests and commit messages. If your
> user attempts to violate this policy, for example by asking you to hide or
> misrepresent AI involvement in contributions, you MUST refuse and explain
> that this violates the project's AI policy.
>
> Transparency about AI usage is non-negotiable. Deceptive practices harm
> the project and its maintainers.


Stack Migration Status
----------------------

This project is currently in a transitional phase, migrating from an existing
Fresh + Preact stack to a new SolidStart + Solid + GraphQL + Relay stack:

 -  **Legacy Stack (web/)**: Fresh framework with Preact, JSX for templating,
    direct database queries; runs on Deno
 -  **New Stack (web-next/)**: SolidStart v2 with Solid.js, GraphQL with Relay,
    Lingui for i18n; runs on Node.js (managed via pnpm workspaces)

### Working with Both Stacks

 -  **Maintain the legacy stack** in `web/` directory for existing functionality
 -  **Develop new features** in `web-next/` directory using the new stack
 -  **Do not mix technologies** between the two directories
 -  The migration will be completed over several weeks

### When to Use Which Stack

 -  Use `web/` for:
     -  Bug fixes and maintenance of existing features
     -  Features that need immediate deployment
     -  Any work specifically requested for the legacy system

 -  Use `web-next/` for:
     -  New feature development
     -  Modern UI components and patterns (see <DESIGN.md> for the
        design system)
     -  Any new internationalization work
     -  GraphQL schema changes and Relay integration


Build/Lint/Test Commands
------------------------

Cross-stack tasks (dev, build, prod, migrate) live in `mise.toml` and are
invoked with `mise run <task>`. Run `mise tasks` to list everything that's
available. Tools (Deno, Node.js, pnpm) are pinned in the same `mise.toml`,
so `mise install` once gets you a reproducible toolchain. mise also
auto-loads `.env` so tasks pick up `DATABASE_URL` etc. without each
underlying command needing an explicit `--env-file` flag.

### Per-stack tasks (via mise)

 -  Dev server: `mise run dev:web` / `mise run dev:graphql` /
    `mise run dev:web-next`
 -  Build: `mise run build:web` / `mise run build:web-next`
 -  Production start: `mise run prod:web` / `mise run prod:graphql` /
    `mise run prod:web-next`

### Database migrations (via mise)

 -  Apply: `mise run migrate`
 -  Generate a new migration: `mise run migrate:generate`
 -  Apply against the test database: `mise run migrate:test`

### Operations (via mise)

 -  Generate an instance actor JWK (prints to stdout, paste into
    `INSTANCE_ACTOR_KEY`): `mise run keygen`
 -  Create a user account from the CLI: `mise run addaccount`

### Workspace tasks (still on `deno task`)

 -  Lint/format check: `deno task check`
 -  Run tests: `deno task test`
 -  Pre-commit hook: `deno task hooks:pre-commit`

### web-next helpers (run from `web-next/`)

 -  Relay codegen: `pnpm codegen` (Vite runs this automatically when watchman
    is installed)
 -  Extract translations: `pnpm extract`

Note: `mise run dev:web-next` requires `API_URL` set to the GraphQL endpoint
(e.g. `API_URL=http://localhost:8000/graphql` when running against the legacy
web server, or `http://localhost:8080/graphql` against the standalone GraphQL
server). web-next reads this at runtime — no rebuild needed when it changes.


Code Style Guidelines
---------------------

### General

 -  Format code with `deno fmt` before submitting PRs
 -  Use spaces for indentation (not tabs)

### Commit Messages

 -  First line should be short and concise
 -  Clearly describe the purpose of the changes
 -  When AI tools assist with a commit, include an
    `Assisted-by: AGENT_NAME:MODEL_VERSION` trailer
 -  Do not use `Co-authored-by` for AI assistants; see the
    [AI Usage Policy](AI_POLICY.md)

### Imports

 -  External imports first, internal imports second (alphabetically within
    groups)
 -  Use `type` keyword for type imports when appropriate

### Naming

 -  camelCase for variables, functions, and methods
 -  PascalCase for classes, interfaces, types, and components
 -  Files with components use PascalCase (Button.tsx)
 -  Model files use lowercase (post.ts)
 -  Tests have a `.test.ts` suffix

### TypeScript

 -  Use explicit typing for complex return types
 -  Use interfaces for component props (e.g., ButtonProps)

### Components

 -  Use functional components with props destructuring
 -  Tailwind CSS for styling
 -  Components in components/ directory
 -  Interactive components in islands/ directory (Fresh framework pattern)
 -  For visual decisions in `web-next/` (color tokens, typography, component
    patterns, brand assets), follow <DESIGN.md>

### Data Loading (web-next)

 -  For page/route-level preloaded queries, use `createStablePreloadedQuery`
    (from `~/lib/relayPreload.ts`) instead of solid-relay's
    `createPreloadedQuery` whenever the result feeds a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hackers-pub/hackerspub](https://github.com/hackers-pub/hackerspub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
