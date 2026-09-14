---
trigger: always_on
description: An MCP server (`@antonytm/mcp-sitecore-server`) that gives agents read/write access to
---

# mcp-sitecore-server

An MCP server (`@antonytm/mcp-sitecore-server`) that gives agents read/write access to
Sitecore over four independent API **surfaces**: the **Authoring and Management GraphQL
API**, the **Item Service** (REST/SSC), **GraphQL Edge/preview**, and **Sitecore PowerShell
Extensions (SPE) remoting**. It targets both SitecoreAI and XM/XP, so treat every surface as
optional and let the gating decide — see tool gating below.

## Tests

`package.json` holds the scripts. Two things it does not say:

- `npm run test:unit` needs no Sitecore instance and should always pass. `npm test` builds,
  bundles and runs the full suite, which needs a CM in `.env` with SPE Remoting enabled —
  but **no seeded content**: every live file creates what it needs and deletes it again
  (`tests/fixtures.ts`).
- The `--config vitest.unit.config.ts` flag scopes to `tests/unit`; both configs map the
  `@/` alias, so the unit files collect under either.

```shell
npx vitest run --config vitest.unit.config.ts tests/unit/projection.test.ts
npx vitest run --config vitest.unit.config.ts -t "name of the case"
```

**Writing a live test:** call `seedScratch(label, [names])` at the top of the file and
register `afterAll(() => scratch.cleanup())`. That creates `/sitecore/content/MCP-<label>-<unique>`
and a matching template, and the cleanup takes both, plus anything the test archived on its
way through. The other fixtures cover what a family needs on top: `seedPresentation` /
`applyPresentation` for layouts and renderings, `seedUser` / `seedRole` for accounts,
`assignWorkflow` / `addWorkflowEvent` for workflow state, `ensureLanguage` for a second
language (**a different locale per file** — files run in parallel, and a shared language is
deleted out from under the file still using it), plus `seedTemplate`, `seedChild` and
`linkItems`.

Assert against the seed, not against a constant: `scratch.template.name`, never
`"Sample Item"`. The two things a new live test gets wrong most often are that the tools
return the **projected** shape (`ID`, not `ID.ToString`; nine fields on an account, not the
whole `User` graph — see `projection.ts`), and that SPE reports a missing item as an *error
result* rather than an empty one, so check `isError` instead of parsing.

`vitest.config.ts` caps the live suite at four worker processes and retries once: 159 files
each spawning a server against one CM is what makes an unconstrained run flaky, not the
tests themselves.

Read the failure signature before blaming a change: `Unexpected token 'G', "Get-Item …"`
means SPE returned its own error text rather than JSON, usually for an item the test
expected to exist; `rejected its arguments before reaching Sitecore` means the test's
arguments do not match the schema; `Login failed: 403` / `came from Auth0` is configuration;
`fixture script failed after 3 attempts` is the CM refusing the seed, not the test.

Runtime config comes from `.env` (copy `.env.template`). CI runs lint, typecheck+build,
bundle, unit tests, and a non-blocking `npm audit`.

## Lint

`npm run lint` is ESLint 10 with typescript-eslint's non-type-aware recommended set; the
type-aware pass is `npm run typecheck`, so the two do not overlap. Two things that are
easy to undo by accident:

- The compiler is pinned to TypeScript **6**, the last release with the JS API, because
  typescript-eslint cannot load against typescript@7's native compiler — its `typescript`
  entry point exports a version string where the API used to be. Bumping to 7 breaks
  `npm run lint`, not the build; see typescript-eslint/typescript-eslint#10940.
- `no-explicit-any` is a **warning**: 124 remain across four loosely-typed remote surfaces,
  and turning it into an error would fail CI on debt that predates the linter. Errors are
  the count that must stay at zero.

## Architecture

**Startup:** `index.ts` reads `TRANSPORT` and calls `stdio.ts` or `streamable-http.ts`;
both build the server via `server.ts:getServer(config)`. (`src/run.ts`, behind `npm run
run`, is a scratch harness for poking the PowerShell client — not an entry point.)
`config.ts` parses env into a `Config` at import time and exports `redactConfig` — every
path that exposes config to a client (the `config` tool, the `config://main` resource) must
go through it.

**Registration pipeline** (`register.ts`): `TOOL_GROUP_REGISTRARS` maps each group name to
an ordered list of registrar functions `(server, config) => void`. `registerAll` skips
whole groups when gated, so a disabled group also skips its registrars' startup cost
(schema introspection, index lookups). Adding a tool = write the registrar, import it, add
it to its group's array.

**Tool gating** (`tool-profiles.ts`): `TOOL_GROUPS` (allowlist of groups), `DISABLED_TOOLS`
(denylist of exact names), `TOOL_PROFILE` (named `no-*` presets, unioned; denylist wins).
Gating is installed on the server *before* the first `registerTool`, and
`registerGuides` is gated on the same object so every offered guide has the tools its
steps name. The group names are the directory layout under `src/tools/`,
not a separate taxonomy.

**Guides** (`src/guides/`): three procedures served as `guide://` resources by

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Antonytm/mcp-sitecore-server](https://github.com/Antonytm/mcp-sitecore-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
