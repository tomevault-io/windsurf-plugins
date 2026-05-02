---
trigger: always_on
description: Provides context sources for fetching and extracting content from linked
---

Guidance for LLM-based code agents
==================================

This file provides guidance to LLM-based code agents (e.g., Claude Code,
OpenCode) when working with code in this repository.


Project overview
----------------

Vertana is an LLM-powered agentic translation library for TypeScript/JavaScript.
It uses autonomous agent workflows to gather contextual information for
high-quality translations that preserve meaning, tone, and formatting.
The library uses the [Vercel AI SDK] (*ai* package) for LLM interactions.

[Vercel AI SDK]: https://sdk.vercel.ai/


Development commands
--------------------

This is a polyglot monorepo supporting Deno, Node.js, and Bun.
Use [mise] to manage runtime versions.

[mise]: https://mise.jdx.dev/

### Package manager

This project uses Deno as the primary development tool and pnpm for
npm-related tasks (building for npm publishing).

> [!IMPORTANT]
> Do *not* use npm or Yarn as package managers in this project.  Always use
> Deno tasks (`deno task ...`) for development workflows and pnpm
> (`pnpm run ...`) only for npm build tasks.

### Installation

~~~~ bash
mise run install
~~~~

### Quality checks

~~~~ bash
mise run check  # Type check, lint, format check, and dry-run publish
mise run fmt    # Format code
deno lint       # Run linter
~~~~

### Testing

~~~~ bash
mise run test:deno   # Run tests with Deno (requires .env.test file)
mise run test:node   # Run tests with Node.js
mise run test:bun    # Run tests with Bun
mise run test        # Run all checks and tests across all runtimes
~~~~

### Building (for npm publishing)

~~~~ bash
pnpm run -r build        # Build all packages with tsdown
~~~~

### Version management

All packages must share the same version.  Use the check-versions script:

~~~~ bash
mise run check-versions          # Check for version mismatches
mise run check-versions --fix    # Auto-fix version mismatches
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
>  -  *package.json*: Add to `dependencies` or `devDependencies` (for Node.js/Bun)
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

Using the project-local *tmp/* directory allows you to import `@vertana/*`
packages with relative imports, whereas using the system */tmp* would require
absolute paths since it is outside the workspace.


Architecture
------------

### Package structure

 -  *@vertana/core* (*packages/core/*): Core translation logic and utilities.
    Contains chunking, evaluation, refinement, selection, and translation
    orchestration (`translateChunks`).
 -  *@vertana/facade* (*packages/facade/*): High-level facade for translation
    tasks.  Contains the main `translate()` function API, which wraps
    the core functionality with a simple interface.
 -  *@vertana/context-web* (*packages/context-web/*): Web context gathering.
    Provides context sources for fetching and extracting content from linked
    web pages.
 -  *@vertana/cli* (*packages/cli/*): Command-line interface for translation.

### Dual publishing

Each package is published to both JSR (Deno) and npm (Node.js/Bun):

 -  JSR uses *deno.json* with TypeScript source directly
 -  npm uses *package.json* with tsdown-built *dist/* output (ESM + CJS + .d.ts)

When adding subpath exports to a package, update the following files:

 -  *deno.json*: Add the subpath to the `exports` field
 -  *package.json*: Add the subpath to the `exports` field
 -  *tsdown.config.ts*: Add the entry point to the build configuration

### Adding new packages

When adding a new package to the monorepo, update the following files:

 -  *README.md* (root): Add the package to the Packages table
 -  *AGENTS.md*: Add the package to the Package structure list (if applicable)
 -  *docs/.vitepress/config.mts*: Add API reference link to `REFERENCES`
 -  *docs/package.json*: Add `"@vertana/<name>": "workspace:"` to `devDependencies`
    (required for Twoslash type checking in documentation)

### Key dependencies

 -  *ai* (Vercel AI SDK): LLM abstraction layer, used via `LanguageModel`
    interface
 -  *@logtape/logtape*: Logging framework
 -  *@standard-schema/spec*: Schema validation interface for library-agnostic
    schema definitions


Development practices
---------------------


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dahlia/vertana](https://github.com/dahlia/vertana) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
