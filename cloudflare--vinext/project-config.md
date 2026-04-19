---
trigger: always_on
description: Instructions for AI agents working on this codebase.
---

# Agent Guidelines

Instructions for AI agents working on this codebase.

---

## Project Overview

**vinext** is a Vite plugin that reimplements the Next.js API surface, with Cloudflare Workers as the primary deployment target. The goal: take any Next.js app and deploy it to Workers with one command.

vinext reimplements the Next.js API surface using Vite, with Cloudflare Workers as the primary deployment target. The goal is to let developers keep their existing Next.js code and deploy it to Workers.

---

## Quick Reference

### Commands

```bash
pnpm test                                        # Vitest — full suite (~2 min, serial)
pnpm test tests/routing.test.ts                  # Run a single test file (~seconds)
pnpm test tests/shims.test.ts tests/link.test.ts # Run specific files
pnpm run test:e2e                                # Playwright E2E tests (all projects, use PLAYWRIGHT_PROJECT=<name> to target one)
pnpm run check                                   # Format, lint, and type checks
pnpm run lint                                    # Lint only (type-aware oxlint)
pnpm run fmt                                     # oxfmt (format)
pnpm run fmt:check                               # oxfmt (check only, no writes)
pnpm run build                                   # Build the vinext package (via vp pack)
```

### Project Structure

```
packages/vinext/src/
  index.ts              # Main Vite plugin
  cli.ts                # vinext CLI
  shims/                # One file per next/* module
  routing/              # File-system route scanners
  server/               # SSR handlers, ISR, middleware
  cloudflare/           # KV cache handler

tests/
  *.test.ts             # Vitest tests
  fixtures/             # Test apps (pages-basic, app-basic, etc.)
  e2e/                  # Playwright tests

examples/               # User-facing demo apps
```

### Key Files

| File                       | Purpose                                                            |
| -------------------------- | ------------------------------------------------------------------ |
| `index.ts`                 | Vite plugin — resolves `next/*` imports, generates virtual modules |
| `shims/*.ts`               | Reimplementations of `next/link`, `next/navigation`, etc.          |
| `server/dev-server.ts`     | Pages Router SSR handler                                           |
| `entries/app-rsc-entry.ts` | App Router RSC entry generator                                     |
| `routing/pages-router.ts`  | Scans `pages/` directory                                           |
| `routing/app-router.ts`    | Scans `app/` directory                                             |

---

## Development Workflow

### Adding a New Feature

1. **Check if Next.js has it** — look at Next.js source to understand expected behavior
2. **Search the Next.js test suite** — before writing code, search `test/e2e/` and `test/unit/` in the Next.js repo for related test files (see below)
3. **Add tests first** — put test cases in the appropriate `tests/*.test.ts` file
4. **Implement in shims or server** — most features are either a shim (`next/*` module) or server-side logic
5. **Add fixture pages if needed** — `tests/fixtures/` has test apps for integration testing
6. **Run the relevant test file(s)** to verify your changes (see [Running Tests](#running-tests) below)

### Searching the Next.js Test Suite

**This is a required step for all feature work and bug fixes.** Before writing code, search the Next.js repo's `test/e2e/` and `test/unit/` directories for tests related to whatever you're working on. Search broadly, not just for exact feature names.

For example, when working on middleware:

- Search for `middleware` and `proxy` in test directory names
- Search for error messages like `"must export"` to find validation tests
- Check for edge cases like missing exports, misspelled names, invalid configs

**Why this matters:** vinext aims to match Next.js behavior exactly. If Next.js has a test for it, we should have an equivalent test. Missing this step has caused silent behavioral differences, like middleware failing open on invalid exports instead of throwing an error (which Next.js tests explicitly).

When you find relevant Next.js tests, port the test cases to our test suite and include a comment linking back to the original Next.js test file:

```ts
// Ported from Next.js: test/e2e/app-dir/proxy-missing-export/proxy-missing-export.test.ts
// https://github.com/vercel/next.js/blob/canary/test/e2e/app-dir/proxy-missing-export/proxy-missing-export.test.ts
```

**Local Next.js clone for fast searching:**

Keep a local clone of the Next.js repo for fast `rg` (ripgrep) searches. This is much faster and more reliable than `gh search code` for exploratory searches.

```bash
# First time: clone into .nextjs-ref (gitignored)
git clone --depth 1 --single-branch --branch canary https://github.com/vercel/next.js.git .nextjs-ref

# Update periodically
git -C .nextjs-ref pull --ff-only
```

The `.nextjs-ref` directory is gitignored. Use it for searching source, tests, and closed PR context:

```bash
# Search test suite for how Next.js handles a behavior
rg -rn "javascript:" .nextjs-ref/test/ --include "*.test.*" -l

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudflare/vinext](https://github.com/cloudflare/vinext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
