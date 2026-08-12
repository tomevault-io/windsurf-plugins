---
trigger: always_on
description: * Be critical and thorough. Prefer truth and direct feedback over politeness
---

# AGENTS.md

## general

* Be critical and thorough. Prefer truth and direct feedback over politeness
   * After every response to me, end it with an emoji
   * Look around and use existing patterns and code when possible. Look for:
      * Similar components and use their patterns
      * Library code you can reuse
      * Existing dependencies from package.json or Cargo.toml that you should use
   * If you see a pattern that is not used, consider adding it, but carefully and
     judiciously
   * Always consider the developer experience:
      * Am I placing a burden on the developer with this change?
      * Is it as easy to use / execute / import / configure as possible?
   * When making _any_ changes:
      * Consider the impact on other parts of the codebase
         * What tests, documentation, etc. needs to be updated?
         * Search for other files that should be changed after what you just did
      * How has the context changed now that I've made this change?
         * Should I refactor the code to introduce an abstraction to make it more
           maintainable?
         * Should I delete anything that's now unused?
   * Check your work after you finish a task:
      * Did I address everything I was asked to?
      * Run `npm run standards` (or `tsc` / `eslint` / `commitlint` / `markdownlint` as
        appropriate)
      * Test significant changes by:
         * Running the tests
         * Running the app and manually testing the changes (Tauri MCP or Playwright MCP)

## Front-End Development

   * Pay attention to the current version of the component, and use a similar pattern as
     set by existing elements
   * Consider accessibility / a11y
   * Create reusable components rather than ad-hoc solutions

## css-scss

* If using a component library, prefer using the component's existing props etc. over
     custom styling
   * If none is available, prefer pre-existing utility classes over custom styling
   * Avoid ad-hoc CSS unless necessary
   * Use BEM or other similar naming conventions for custom CSS
   * Use CSS variables for theme-able values
   * Consider adding a custom utility class to the global CSS/SCSS if it seems
     necessary/used in multiple places

## do-not

* ABSOLUTELY DO NOT `git push` without express permission * ABSOLUTELY DO NOT create
     ad-hoc test scripts. If you absolutely must, clean up those files when you're done
   * ABSOLUTELY DO NOT ignore "pre-existing" TypeScript or linting errors. If you see
     them, fix them before proceeding
   * ABSOLUTELY DO NOT ignore "pre-existing" tests that fail. If you see them, fix them
     before proceeding
   * ABSOLUTELY DO NOT ignore "pre-existing" documentation that is out of date. If you see
     it, fix it before proceeding
   * ABSOLUTELY DO NOT use `@deprecated` on anything unless you are explicitly asked to.
     Always fully refactor and delete old code as-needed instead of deprecating it
   * ABSOLUTELY DO NOT implement functionality that already exists in a library or
     package, especially if that package is already installed in the project * Examples:
     parsing, validation, formatting
   * ABSOLUTELY DO NOT disable linting rules (ESLint, oxlint, clippy, etc.) in the config
     to get around linting errors. Fix the underlying issues
   * ABSOLUTELY DO NOT instruct me to do things like "run the dev server and test it out,"
     "run the tests," "install this module", or anything else that you can do yourself as
     part of the task

## git

* Follow these rules when running git commands and making commits:
      * https://raw.githubusercontent.com/silvermine/silvermine-info/refs/heads/master/commit-history.md
      * https://raw.githubusercontent.com/silvermine/standardization/refs/heads/master/commitlint.js

## mcps

* Always use context7 when I need code generation, setup or configuration steps, or
     library/API documentation. This means you should automatically use the Context7 MCP
     tools to resolve library ID and get library docs without me having to explicitly ask.
   * If you make UI changes, use MCP tools to test them in a real environment unless
     project-specific rules say not to
      * Use the Tauri MCP when working within a Tauri app
      * Use Playwright for other projects

## npm

* Always use `npm` instead of `pnpm` or `yarn`
   * Always use the --save-exact flag when installing a dependency
   * Use the `-y` flag with `npx` when running a command

## releasing

* When a task touches release prep or release automation, follow `RELEASING.md`.
   * Keep `package.json`, `Cargo.toml`, `crates/device-ai/Cargo.toml`, and the root
     `device-ai` dependency version aligned.
   * Run `npm run release:validate -- vX.Y.Z` before creating a release tag.
   * Releases publish only from GitHub after a verified signed `v*` tag push.

## testing

* When writing tests, prefer practical e2e tests over unit tests, but add unit tests
     for critical functionality or complex logic
   * If you write tests, always run them

## typescript-javascript

Follow these standards:

   * https://github.com/silvermine/silvermine-info/raw/refs/heads/master/coding-standards/typescript.md

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hypothesi/tauri-plugin-device-ai-apis](https://github.com/hypothesi/tauri-plugin-device-ai-apis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
