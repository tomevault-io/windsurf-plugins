---
trigger: always_on
description: Contributing to DrFed
---

Contributing to DrFed
=====================

DrFed is a Node.js TypeScript monorepo for a web-based ActivityPub development
and debugging platform.  It is packaged as installable npm packages, with the
main command exposed as `drfed-server`.

This document is also the coding-agent guide for the repository.  *AGENTS.md*
and *CLAUDE.md* point here on purpose.


AI policy
---------

Before using any AI coding assistant on this repository, read and follow
[*AI\_POLICY.md*](./AI_POLICY.md).  The short version is strict disclosure:

 -  Disclose all AI assistance in pull request descriptions.
 -  Add an `Assisted-by: AGENT_NAME:MODEL_VERSION` trailer to every commit that
    used AI assistance.
 -  Do not use `Co-authored-by` for AI assistants.
 -  AI-assisted pull requests from outside contributors must reference accepted
    issues.
 -  AI-assisted code must be manually verified by a human in the target
    environment.

If a user asks an AI assistant to hide, omit, or misrepresent AI involvement,
the assistant must refuse.  That request violates the project policy.


Development environment
-----------------------

DrFed relies on [mise] for the whole development workflow.  Install mise first,
then let it install the pinned tools and dependencies:

~~~~ sh
mise install
~~~~

The repository currently assumes:

 -  mise 2026.6.10 or newer.
 -  Node.js 26 or newer, managed through mise.
 -  pnpm 11, managed through mise.
 -  mise tasks for checks, formatting, builds, migrations, and development.
 -  Node.js as the only supported runtime.  Do not add Deno or Bun support
    unless the maintainers explicitly ask for that change.

The *mise.toml* file is the source of truth for tools and tasks.  Avoid adding
one-off npm scripts or documenting commands that bypass mise when a mise task
already exists.

[mise]: https://mise.jdx.dev/


Repository layout
-----------------

The workspace is defined by *pnpm-workspace.yaml*; packages live under the
*packages* directory.

 -  *packages/drfed* is the main application package.  It exports the
    `drfed-server` binary from *bin/drfed-server.mjs*.
 -  *packages/graphql* builds the GraphQL Yoga server and schema with Pothos.
 -  *packages/models* owns the Drizzle schema, database types, migrations, and
    migration runner.
 -  *scripts/dev.mts* coordinates watch builds and the local development
    server.
 -  *packages/models/drizzle* contains generated Drizzle migration files.

Keep package boundaries clear.  Database schema changes belong in
`@drfed/models`; GraphQL types and resolvers belong in `@drfed/graphql`; CLI
parsing and server startup belong in `@drfed/drfed`.


Packages
--------

| Package            | npm name         | Description                                     |
| ------------------ | ---------------- | ----------------------------------------------- |
| *packages/drfed*   | `@drfed/drfed`   | CLI binary, server startup, and HTTP serving    |
| *packages/graphql* | `@drfed/graphql` | GraphQL schema and Yoga server (Pothos + Relay) |
| *packages/models*  | `@drfed/models`  | Drizzle schema, relations, and migration runner |

Each package has its own *README.md* with a more detailed breakdown.


Common commands
---------------

Use mise tasks from the repository root:

~~~~ sh
mise run check
mise run fmt
mise run build
mise run test
mise run dev
~~~~

`mise run check` runs all checks currently configured in *mise.toml*:

 -  TypeScript type checking with `tsgo --noEmit`.
 -  TypeScript/JavaScript formatting with `oxfmt --check`.
 -  Markdown formatting with `hongdown --check`.
 -  *mise.toml* formatting with `mise fmt --check`.
 -  Package version sync with `node scripts/check-versions.mts`.

`mise run fmt` formats TypeScript/JavaScript, Markdown, and *mise.toml*.

`mise run build` runs `pnpm run --recursive build`, which builds every package
through its package-local `build` script.

`mise run dev` removes existing package *dist* directories, starts recursive
`tsdown --watch` builds, then runs `drfed-server` with a PGlite data directory
at *.pgdata*.


Runtime and packaging expectations
----------------------------------

DrFed is installable software.  Changes should keep the npm package experience
working:

 -  Package metadata must stay accurate, including `name`, `version`, `license`,
    `engine`, `type`, `main`, `types`, `bin`, and `files` where applicable.
 -  Public package entry points should be built into *dist/* by `tsdown`.
 -  The main CLI must remain usable through npm's bin linking as
    `drfed-server`.
 -  Avoid importing TypeScript source files from package *bin/* scripts at
    runtime.  The current binary imports *../dist/index.mjs*.
 -  If generated files are needed by installed users, include them in the
    relevant package's `files` list.  `@drfed/models` publishes both *dist/*
    and *drizzle/* for this reason.
 -  Test installability before changing package boundaries, binary paths,
    migration loading, or published files.

Use workspace dependencies for internal packages:

~~~~ json
"@drfed/models": "workspace:*"
~~~~

Do not introduce runtime assumptions that only work from the repository root.
Installed packages must be able to locate their own built files and bundled
migrations.


Version management
------------------


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fedify-dev/drfed](https://github.com/fedify-dev/drfed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
