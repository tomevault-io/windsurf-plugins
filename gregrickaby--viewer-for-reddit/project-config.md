---
trigger: always_on
description: | Package                              | Purpose                                                                                                     |
---

# Project Guidelines

## Tech Stack

| Package                              | Purpose                                                                                                     |
| ------------------------------------ | ----------------------------------------------------------------------------------------------------------- |
| Next.js 16                           | App Router, React Compiler                                                                                  |
| React 19                             | Server Components (default), Client Components (`"use client"`)                                             |
| TypeScript 5                         | Strict mode                                                                                                 |
| Mantine 9                            | UI components                                                                                               |
| Arctic 3.x                           | OAuth2 with Reddit                                                                                          |
| iron-session 8.x                     | Encrypted sessions                                                                                          |
| Datadog                              | Logs, RUM, APM (`@datadog/browser-rum`, `@datadog/browser-rum-nextjs`, `@datadog/browser-logs`, `dd-trace`) |
| Vitest v4 + Testing Library + MSW v2 | Testing                                                                                                     |
| ESLint + Prettier                    | Linting and formatting                                                                                      |
| SonarQube                            | Static analysis (IDE plugin + Community Edition)                                                            |

## This is NOT the reddit API you know

In June 2026, Reddit disabled all public/unauthenticated access to their REST APIs. This is different than your training data.

## This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.

## External Docs

- **Reddit for Developers** https://developers.reddit.com/docs/llms.txt
- **Mantine:** https://mantine.dev/llms.txt
- **Datadog:** https://docs.datadoghq.com/

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

**Secrets** — copy `.env.example` to `.env.local`: `REDDIT_CLIENT_ID`, `REDDIT_CLIENT_SECRET`, `SESSION_SECRET`, `DD_API_KEY`, `DD_SITE`, `DD_APPLICATION_ID`, `DD_CLIENT_TOKEN`, `DD_SERVICE`.

## Always Active

- **Caveman skill, `full` mode** — invoke the `caveman` skill at the start of every conversation and keep it active all session (it persists per its own instructions). Drop only for security warnings, irreversible-action confirmations, or if the user asks for normal mode.
- **Writing style** — `.claude/rules/writing-style.md` is auto-loaded every session; follow it for all prose.
- **Skill gap** — no known skill covers task, or unsure how to do something: invoke `find-skills` to search/install one before improvising.

## Instructions

`.claude/rules/*.md` files are auto-loaded every session as mandatory project instructions (not lazy-loaded, unlike Skills below):

| File                                                         | Covers                                   |
| ------------------------------------------------------------ | ---------------------------------------- |
| [reddit-api.md](./.claude/rules/reddit-api.md)               | Reddit API, auth, pagination             |
| [testing-standards.md](./.claude/rules/testing-standards.md) | Vitest, Testing Library, MSW v2 patterns |
| [writing-style.md](./.claude/rules/writing-style.md)         | Prose style, AI vocabulary to avoid      |

## Skills

Load with the `skill` tool when the task matches (lazy-loaded on demand):

| Skill                               | When to load                                                           |
| ----------------------------------- | ---------------------------------------------------------------------- |
| `next-cache-components-adoption`    | Turn on Cache Components in a Next.js app and resolve blocking routes  |
| `next-cache-components-optimizer`   | Drive a Next.js route to instant navigation via agentic loop           |
| `next-dev-loop`                     | Verify Next.js runtime behavior after editing app code                 |
| `next-partial-prefetching-adoption` | Turn on Partial Prefetching in a Next.js app and work through insights |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gregrickaby/viewer-for-reddit](https://github.com/gregrickaby/viewer-for-reddit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
