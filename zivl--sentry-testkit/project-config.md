---
trigger: always_on
description: Instructions for AI coding agents (Claude Code, OpenAI Codex, Gemini CLI, and others) working in this repository. This is the **single canonical instructions file** — `CLAUDE.md` only references this one.
---

# AGENTS.md

Instructions for AI coding agents (Claude Code, OpenAI Codex, Gemini CLI, and others) working in this repository. This is the **single canonical instructions file** — `CLAUDE.md` only references this one.

> Also read [CONTRIBUTING.md](CONTRIBUTING.md) for commit conventions, git workflow, and release details — it takes precedence on those topics.

---

## Project Overview

**sentry-testkit** is a testing utility library. It intercepts Sentry SDK error/performance reports during tests, stores them in memory, and exposes an API for test assertions. Reports never reach Sentry's servers.

This is a **Yarn 4 monorepo**. The main package is `packages/sentry-testkit/`. There is also an Expo/React Native test app under `apps/`.

---

## Setup

```bash
git clone git@github.com:zivl/sentry-testkit.git
cd sentry-testkit
yarn install
```

**Node version**: use whatever `.nvmrc` / `.node-version` specifies, or the `engines` field in `package.json`.

---

## Commands

Run from the **repo root** unless noted:

```bash
yarn build          # Build the sentry-testkit package
yarn test           # Run all tests for sentry-testkit
yarn test:expo      # Run tests for the Expo/React Native app
yarn lint           # Lint the sentry-testkit package

# From packages/sentry-testkit/ — narrower test runs:
yarn test -- <file-pattern>                        # e.g. node.test.ts
yarn test -- --testNamePattern="<test name>"       # filter by test name
```

Always run `yarn test` before finishing any change that touches `packages/sentry-testkit/`.

---

## Architecture

The library provides multiple integration modes:

| Mode | Entry file | Mechanism |
|------|-----------|-----------|
| Node.js / Browser | `index.ts` / `browser.ts` | Custom Sentry transport replaces the HTTP sender |
| Local Server | `localServerApi.ts` | Express server that mimics Sentry's API; returns a local DSN |
| Puppeteer | `testkit.ts` | Intercepts page network requests via Puppeteer's Page API |
| Network Interceptor | `initNetworkInterceptor.ts` | Callback hook for manual capture (e.g. nock) |
| Jest Mock | `jestMock.ts` | Convenience wrapper; injects testkit into `global.testkit` |

### Data flow

```
Sentry SDK call
  → sentryTransport.ts   (intercept)
  → parsers.ts           (parse raw envelope / event)
  → transformers.ts      (convert to Report / Transaction)
  → testkit.ts           (store in memory arrays)
  → test assertions      (reports(), transactions(), getExceptionAt(), …)
```

### Key source files (`packages/sentry-testkit/src/`)

| File | Purpose |
|------|---------|
| `index.ts` | Main entry; exports `create()` → `{ sentryTransport, testkit, initNetworkInterceptor, localServer }` |
| `browser.ts` | Same as index but without Node.js / Express imports |
| `testkit.ts` | Core in-memory store + Puppeteer integration |
| `sentryTransport.ts` | Transport adapter (envelope-based; supports Sentry v9/v10) |
| `parsers.ts` | Parses raw Sentry envelopes (multi-item, length-prefixed payloads) |
| `transformers.ts` | Converts parsed items into typed `Report` / `Transaction` / `Log` objects |
| `localServerApi.ts` | Express handlers for `/api/{project}/store/` and `/api/{project}/envelope/` |
| `types.ts` | All public TypeScript interfaces (`Testkit`, `Report`, `Transaction`, `Span`, …) |

### Test files (`packages/sentry-testkit/__tests__/`)

| File | What it covers |
|------|---------------|
| `commonTests.ts` | Shared suite run by node, browser, and react tests |
| `node.test.ts` | Node.js integration |
| `browser.test.ts` | Browser (jsdom) integration |
| `react.test.tsx` | React integration |
| `puppeteer.test.ts` | Puppeteer integration |
| `local-server.test.ts` | Local server integration |
| `network-interception.test.ts` | Network interceptor |
| `jest-mock.test.ts` | Jest mock helper |
| `parsers.test.ts` | Envelope parser unit tests |
| `logs.test.ts` | Structured logs capture |

---

## Coding Conventions

- **TypeScript** throughout. No `any` unless absolutely unavoidable.
- Prefer `const` over `let`; avoid `var`.
- Functions should have minimal side effects.
- Use clear, descriptive variable and function names.
- Do not add comments that explain *what* code does — name things well instead. Only comment *why* when the reason is non-obvious (a hidden constraint, a workaround for a specific bug).
- Do not add error handling or fallbacks for scenarios that cannot happen; trust TypeScript types and framework guarantees.
- Do not introduce abstractions beyond what the task requires.

---

## Git Workflow

- **Rebase, never merge** when integrating with `master` (`git rebase origin/master`, not `git merge`).
- **Squash commits** — each PR should land as a single, well-formed commit on `master`.
- **No attribution trailers** — commit messages and PR descriptions must not contain `Co-Authored-By`, "Generated with", or any other AI-attribution lines. Commits are authored solely by the repository owner.
- Commit messages must follow **Conventional Commits**:
  ```
  <type>[optional scope]: <description>

  [optional body]

  [optional footer]
  ```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zivl/sentry-testkit](https://github.com/zivl/sentry-testkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
