---
trigger: always_on
description: Guidelines for LLM-powered code assistants
---

Guidelines for LLM-powered code assistants
==========================================

This file provides guidance to LLM-powered code assistants when working
with code in this repository.


Project overview
----------------

BotKit is a TypeScript framework for creating ActivityPub bots, built on top
of Fedify.  It supports both Deno and Node.js environments and provides a
simple API for creating standalone ActivityPub servers that function as bots.


Development commands
--------------------

### Primary commands (Deno-based)

 -  `deno task check` - Full codebase validation (type check, lint, format
    check, publish dry-run, version check)
 -  `deno task test` - Run Deno tests with network access to hollo.social
 -  `deno task test:node` - Run Node.js tests via pnpm
 -  `deno task test-all` - Run all checks and tests (check + test + test:node)
 -  `deno task coverage` - Generate test coverage report in HTML format

### Build commands

 -  `pnpm build` - Build via npm script (runs tsdown)
 -  `pnpm test` - Run Node.js tests after installing dependencies

### Code quality

 -  `deno lint` - Lint TypeScript code
 -  `deno fmt` - Format code (excludes .md, .yaml, .yml files)
 -  `deno fmt --check` - Check code formatting without modifying files
 -  `deno check src/` - Type check source files

### Adding dependencies

When adding new dependencies, always check for the latest version:

 -  *npm packages*: Use `npm view <package> version` to find the latest version
 -  *JSR packages*: Use the [JSR API] to find the latest version

Always prefer the latest stable version unless there is a specific reason
to use an older version.

> [!IMPORTANT]
> Because this project supports both Deno and Node.js, dependencies must
> be added to *both* configuration files:
>
>  -  *deno.json*: Add to the `imports` field (for Deno)
>  -  *package.json*: Add to `dependencies` or `devDependencies` (for Node.js)
>
> Forgetting to add a dependency to *package.json* will cause Node.js tests
> to fail with `ERR_MODULE_NOT_FOUND`, even if Deno tests pass.

[JSR API]: https://jsr.io/docs/api


Architecture
------------

### Core module structure

 -  *src/mod.ts* - Main entry point, re-exports all public APIs
 -  *src/bot.ts* - Core Bot interface and createBot function
 -  *src/bot-impl.ts* - Internal Bot implementation
 -  *src/session.ts* - Session management for bot operations
 -  *src/message.ts* - Message types and ActivityPub objects (Note, Article,
    etc.)
 -  *src/events.ts* - Event handler type definitions
 -  *src/text.ts* - Text formatting utilities (mention, hashtag, link, etc.)
 -  *src/emoji.ts* - Custom emoji handling
 -  *src/reaction.ts* - Like and reaction implementations
 -  *src/repository.ts* - Data storage abstractions
 -  *src/follow.ts* - Follow request handling

### Key concepts

 -  *Bot*: The main bot instance created with `createBot()`, handles events
    and provides session access
 -  *Session*: Scoped bot operations for publishing content and managing state
 -  *Message*: ActivityPub objects like Note, Article, Question with rich text
    support
 -  *Repository*: Storage backend abstraction (Memory, KV-based, cached
    variants)
 -  *Event Handlers*: Functions for responding to ActivityPub activities
    (mentions, follows, likes, etc.)

### Build system

 -  Uses *tsdown* for cross-platform builds (Deno -> Node.js/npm)
 -  Generates ESM (_dist/\*.js_) and CommonJS (_dist/\*.cjs_) outputs
 -  Creates TypeScript definitions for both (_dist/\*.d.ts_, _dist/\*.d.cts_)
 -  Includes Temporal polyfill injection for Node.js compatibility

### Dual runtime support

 -  Primary development in Deno with *deno.json* configuration
 -  Node.js support via *package.json* and tsdown transpilation
 -  Separate import maps for each runtime (JSR for Deno, npm for Node.js)


Development practices
---------------------

### Test-driven development

This project follows test-driven development (TDD) practices:

 -  *Write tests first*: Before implementing new functionality, write tests
    that describe the expected behavior.  Confirm that the tests fail before
    proceeding with the implementation.
 -  *Regression tests for bugs*: When fixing bugs, first write a regression
    test that reproduces the bug.  Confirm that the test fails, then fix the
    bug and verify the test passes.

### Running tests

 -  Deno tests: `*.test.ts` files, run with `deno task test`
 -  Node.js tests: Built output tested in *dist/* directory with Node's
    built-in test runner
 -  Coverage reports available via `deno task coverage`

Always run the full test suite with `deno task test-all` to ensure both Deno
and Node.js compatibility.

### When making changes

1.  Run `deno task check` before committing to validate all aspects
2.  The build process (*tsdown*) generates dual outputs for both runtimes
3.  Tests should work in both Deno and Node.js environments
4.  *Update documentation*: New features must be documented in the *docs/*
    directory
5.  *Update changelog*: Any user-facing changes must be recorded in
    *CHANGES.md*

### Commit messages

 -  Do not use Conventional Commits (no `fix:`, `feat:`, etc. prefixes).
    Keep the first line under 50 characters when possible.

 -  Focus on *why* the change was made, not just *what* changed.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fedify-dev/botkit](https://github.com/fedify-dev/botkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
