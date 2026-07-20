---
trigger: always_on
description: | Package                              | Purpose                                                                      |
---

# Project Guidelines

## Tech Stack

| Package                              | Purpose                                                                      |
| ------------------------------------ | ---------------------------------------------------------------------------- |
| Next.js 16                           | App Router, React Compiler                                                   |
| React 19                             | Server Components (default), Client Components (`"use client"`)              |
| TypeScript 5                         | Strict mode                                                                  |
| Mantine 9                            | UI components                                                                |
| Arctic 3.x                           | OAuth2 with Reddit                                                           |
| iron-session 8.x                     | Encrypted sessions                                                           |
| Axiom                                | Structured logging (`@axiomhq/logging`, `@axiomhq/nextjs`, `@axiomhq/react`) |
| Vitest v4 + Testing Library + MSW v2 | Testing                                                                      |
| ESLint + Prettier                    | Linting and formatting                                                       |
| SonarQube                            | Static analysis (IDE plugin + Community Edition)                             |

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.

## External Docs

- **Reddit for Developers** https://developers.reddit.com/docs/llms.txt
- **Mantine:** https://mantine.dev/llms.txt
- **Axiom:** https://axiom.co/docs/llms.txt

## Commands

```bash
npm run validate      # Format + typecheck + lint — REQUIRED before completion
npm test              # Run tests
npm run test:coverage # Coverage report
npm run test:watch    # Run tests in watch mode
npm run test:ui       # Run tests with Vitest UI
npm run build         # Production build
npm run codegen       # Generate types from Reddit API (requires script app auth)
npm run sonar         # SonarQube analysis (~6 min)
```

## Instructions

Instructions:

- Do NOT preemptively load all references - use lazy loading based on actual need
- When loaded, treat content as mandatory instructions that override defaults
- Follow references recursively when needed

| File                                                                                          | Covers                                   |
| --------------------------------------------------------------------------------------------- | ---------------------------------------- |
| [reddit-api.instructions.md](./.agents/instructions/reddit-api.instructions.md)               | Reddit API, auth, pagination             |
| [testing-standards.instructions.md](./.agents/instructions/testing-standards.instructions.md) | Vitest, Testing Library, MSW v2 patterns |
| [writing-style.instructions.md](./.agents/instructions/writing-style.instructions.md)         | Prose style, AI vocabulary to avoid      |

## Skills

Load with the `skill` tool when the task matches:

| Skill                           | When to load                                                              |
| ------------------------------- | ------------------------------------------------------------------------- |
| `implement-github-issue`        | User pastes a GitHub issue URL or says "implement issue #N"               |
| `improve-codebase-architecture` | Refactoring, testability, module consolidation                            |
| `next-best-practices`           | File conventions, RSC boundaries, data patterns, metadata, error handling |
| `next-cache-components`         | PPR, `use cache`, `cacheLife`, `cacheTag`                                 |
| `update-instructions`           | After major feature additions or when instructions feel stale             |
| `vercel-react-best-practices`   | React/Next.js performance, bundle optimization                            |

## Core Conventions

**Server Components by default** — only add `"use client"` for interactivity (hooks, events, browser APIs).

**API calls in Server Actions** — `lib/actions/reddit/` (posts, users, subreddits, multireddits, search) and `lib/actions/auth/auth.ts` use `redditFetch<T>()` with `next: {revalidate}`.

**React 19 Compiler** — handles memoization automatically.

**Axiom logging** — `lib/axiom/server.ts` is server-only; use `lib/axiom/client.ts` in Client Components.

**Error tracking** — `instrumentation.ts` wires Axiom's `createOnRequestError` for server-side error logging.

**Route group** — `(shell)` wraps all browsable pages with a shared layout (sidebar, header). Pages outside `(shell)` (about, donate) are standalone.

**Hooks architecture** — `lib/hooks/` contains feature hooks and reusable primitives (`useOptimisticToggle`, `useOptimisticMutation`). All hooks are client-only.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gregrickaby/viewer-for-reddit](https://github.com/gregrickaby/viewer-for-reddit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
