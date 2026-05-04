---
trigger: always_on
description: Provides `createConfigContext()` and `bindConfig()` for config fallbacks.
---

Guidance for LLM-based code agents
==================================

This file provides guidance to LLM-based code agents (e.g., Claude Code,
OpenCode) when working with code in this repository.

> [!IMPORTANT]
> Before performing any tasks, you must read and adhere to the
> [AI usage policy].

[AI usage policy]: ./AI_POLICY.md


Project overview
----------------

Optique is a type-safe combinatorial CLI parser for TypeScript, inspired by
Haskell's [optparse-applicative] and TypeScript's [Zod].  It provides a
functional approach to building command-line interfaces using composable
parsers with full type safety.

This project is hosted on GitHub at [dahlia/optique].

[optparse-applicative]: https://github.com/pcapriotti/optparse-applicative
[Zod]: https://zod.dev/
[dahlia/optique]: https://github.com/dahlia/optique


Development commands
--------------------

This is a polyglot monorepo supporting Deno, Node.js, and Bun.
Use [mise] to manage runtime versions and run development tasks.

[mise]: https://mise.jdx.dev/

### Package manager

This project uses Deno as the primary development tool and pnpm for
npm-related tasks (building for npm publishing).

> [!IMPORTANT]
> Do *not* use npm or Yarn as package managers in this project.  Always use
> mise tasks (`mise run ...` or `mise <task>`) for development workflows.

### Installation

~~~~ bash
mise install  # Install runtime tools (Deno, Node.js, Bun, pnpm)
mise deps     # Install project dependencies
~~~~

### Quality checks

~~~~ bash
mise check       # Type check, lint, format check, and dry-run publish
deno fmt         # Format code
deno lint        # Run linter
~~~~

### Testing

~~~~ bash
mise test:deno   # Run tests with Deno (primary test environment)
mise test:node   # Run tests with Node.js
mise test:bun    # Run tests with Bun
mise test        # Run all checks and tests across all runtimes
~~~~

### Building (for npm publishing)

~~~~ bash
mise build       # Build all packages with tsdown
~~~~

### Version management

All packages must share the same version.  Use the check-versions task:

~~~~ bash
mise check-versions          # Check for version mismatches
mise check-versions --fix    # Auto-fix version mismatches
~~~~

### Adding dependencies

When adding new dependencies, always check for the latest version:

 -  *npm packages*: Use `npm view <package> version` to find the latest version
 -  *JSR packages*: Use the [JSR API] to find the latest version

Always prefer the latest stable version unless there is a specific reason
to use an older version.

> [!IMPORTANT]
> Because this project supports both Deno and Node.js/Bun, dependencies must
> be added to *both* configuration files:
>
>  -  *deno.json*: Add to the `imports` field (for Deno)
>  -  *package.json*: Add to `dependencies` or `devDependencies` (for
>     Node.js/Bun)
>
> For workspace packages, use the pnpm catalog (*pnpm-workspace.yaml*) to manage
> versions centrally.  In *package.json*, reference catalog versions with
> `"catalog:"` instead of hardcoding version numbers.
>
> Forgetting to add a dependency to *package.json* will cause Node.js and Bun
> tests to fail with `ERR_MODULE_NOT_FOUND`, even if Deno tests pass.

[JSR API]: https://jsr.io/docs/api

### Temporary scripts

When creating temporary test scripts, save them in the *tmp/* directory
at the project root (not the system */tmp* directory).  This directory is
already in *.gitignore*.

Using the project-local *tmp/* directory allows you to import `@optique/*`
packages with relative imports, whereas using the system */tmp* would require
absolute paths since it is outside the workspace.


Architecture
------------

### Package structure

 -  *@optique/core* (*packages/core/*): Core parsing library.  Contains parser
    combinators (`parser.ts`), value parsers (`valueparser.ts`), help text
    generation (`usage.ts`), and error handling (`message.ts`).
    This package is pure TypeScript and works in any JavaScript environment.
 -  *@optique/run* (*packages/run/*): CLI integration wrapper.  Provides
    process-integrated `run()` function, argument reading from `process.argv`
    or `Deno.args`, and `process.exit()` handling.
 -  *@optique/config* (*packages/config/*): Configuration file integration.
    Provides `createConfigContext()` and `bindConfig()` for config fallbacks.
 -  *@optique/env* (*packages/env/*): Environment variable integration.
    Provides `createEnvContext()`, `bindEnv()`, and `bool()`.
 -  *@optique/temporal* (*packages/temporal/*): Temporal/Date parsers.
    Provides parsers for date and time values.
 -  *@optique/git* (*packages/git/*): Git reference parsers. Provides async
    value parsers for validating Git references (branches, tags, commits,
    remotes) using isomorphic-git.
 -  *@optique/inquirer* (*packages/inquirer/*): Interactive prompt integration.
    Provides `prompt()` for interactive fallback when CLI values are absent.

### Dual publishing

Each package is published to both JSR (Deno) and npm (Node.js/Bun):

 -  JSR uses *deno.json* with TypeScript source directly
 -  npm uses *package.json* with tsdown-built *dist/* output (ESM + CJS + .d.ts)

When adding subpath exports to a package, update the following files:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dahlia/optique](https://github.com/dahlia/optique) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
