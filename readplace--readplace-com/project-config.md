---
trigger: always_on
description: 1. Install devbox: https://www.jetify.com/docs/devbox/installing-devbox#linux
---

# Development Guidelines

## Setup

1. Install devbox: https://www.jetify.com/docs/devbox/installing-devbox#linux
  a) if unable to install devbox, check [devbox.json](./devbox.json) for required tools and install them manually (Node.js, AWS CLI, Pulumi, etc.), Check [.envrc](./.envrc) for required environment variables and set them in your shell profile (e.g., .bashrc, .zshrc)
1. Run pnpm install to install dependencies

## Product Constraints

### Imports are manual via email

There is no in-app import. Users email Fayner their export file and he runs the import manually (24–48h turnaround — see [pocket-migration.md](./projects/hutch/src/runtime/web/pages/blog/posts/pocket-migration.md) for the user-facing copy). The absence is intentional: do not build a self-serve import flow, do not describe import as a shipped feature without qualifying it's a manual email process, and do not restore the removed "Import Your Data" landing page card.

### Crawler Health Canary Is Load-Bearing

A failure in the [crawler source health canary](./src/packages/crawl-article/scripts/health-sources.js) means production traffic is also blocked for that source. Every entry exists because a real user tried to save that type of URL and the crawler broke on it. When a canary fails, fix the crawler until the canary's URL loads — do not delete the entry to make the workflow green. Removing a source silently accepts that readers will get "Sorry, we couldn't save this link" for any URL matching that edge-sniffer's fingerprint (Cloudflare TLS fingerprinting, Fastly JA3, etc.).

Workflow for a canary failure:
1. Reproduce the failing fetch locally against the same URL before touching any code.
2. Find the block (status code, Cloudflare `cf-mitigated` header, body contents) and pick a mitigation that hits the real origin (HTTP/2 fallback, header tweaks, oembed, etc.).
3. Re-run the canary locally until the failing source passes — never commit until it does.
4. Only then push and watch CI.

## Architecture Guidelines

### Brand & Design Guidelines

For colours, typography, voice, and UI conventions, see the [brand guidelines](./BRAND_GUIDELINES.md).

### Web Adapter Conventions

For HTML/CSS/SSR conventions, see the [web skill](./.claude/skills/web/SKILL.md).

### Test Driven Design

For testing conventions and designing testable code, see the [test-driven-design skill](./.claude/skills/test-driven-design/SKILL.md).

### No Cross-Project Relative Imports

Never import from another project using relative filesystem paths (e.g., `../../../../other-project/dist/...`). These create invisible dependencies that bypass the package manager and risk cyclic imports. Extract shared code into a workspace package instead.

```typescript
// BAD - Invisible sideways dependency
const mod = path.resolve(__dirname, "../../../../hutch/dist/runtime/test-app");
const { createTestApp } = await import(mod);

// GOOD - Declared workspace dependency
const { createTestApp } = await import("hutch-test-app");
```

### Filter and Query Testing Strategy

Use integration tests for comprehensive filter/query functionality testing, not E2E tests. Filter logic tests should verify URL parameters produce correct HTML output using supertest + parseHTML.

| Test concern | Test type | Location |
|-------------|-----------|----------|
| Domain Logic | Unit test | `*.test.ts` next to implementation |
| Web Layer | Integration test | `*.route.test.ts` |


E2E tests have ~11s startup overhead per test (browser, server, navigation). Integration tests avoid this overhead while still testing the full server-side flow.

## Coding Style

### Environment Variable Access

Use `requireEnv` and `getEnv` from [projects/hutch/src/require-env.ts](projects/hutch/src/require-env.ts). Never use `process.env` directly.

```typescript
// BAD - Direct process.env access
const apiKey = process.env.API_KEY;

// GOOD - Required env var (throws if not set)
const apiKey = requireEnv('API_KEY');

// GOOD - Optional env var
const proxyUrl = getEnv('HTTPS_PROXY');
```

**Never default missing environment variables.** Always use `requireEnv` and let the process fail if a variable is not set. Do not use `getEnv` with a fallback (e.g., `getEnv("KEY") ?? ""`) to work around CI environments missing secrets. Instead, ensure the CI environment provides the variable. Silent defaults complicate debugging and create behaviour that takes longer to diagnose.

**Exception:** Playwright config files (`playwright.config.*.ts`) must use `process.env` directly. Importing `getEnv`/`requireEnv` causes the playwright process to load `require-env.ts` outside V8 coverage instrumentation, creating uncovered function entries that break the 100% function coverage threshold.

### Comments Document Why, Not What

Do not add comments that explain what code does. Only add comments to explain **why** when the reasoning isn't obvious.

```typescript
// BAD - Explains what (obvious from code)
// Re-export template function
export { createHomePageContent } from './home.template';

// GOOD - Explains why (not obvious)
// Robots noindex because this page contains personal data
robots: 'noindex, nofollow',
```

When a comment explains a specific line within a block and fits on one line, use an inline comment:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Readplace/readplace.com](https://github.com/Readplace/readplace.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
