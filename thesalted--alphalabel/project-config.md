---
trigger: always_on
description: Use Bun instead of Node.js, npm, pnpm, or vite.
---


Default to using Bun instead of Node.js.

- Use `bun <file>` instead of `node <file>` or `ts-node <file>`
- Use `bun test` instead of `jest` or `vitest`
- Use `bun build <file.html|file.ts|file.css>` instead of `webpack` or `esbuild`
- Use `bun install` instead of `npm install` or `yarn install` or `pnpm install`
- Use `bun run <script>` instead of `npm run <script>` or `yarn run <script>` or `pnpm run <script>`
- Use `bunx <package> <command>` instead of `npx <package> <command>`
- Bun automatically loads .env, so don't use dotenv.

## APIs

- `Bun.serve()` supports WebSockets, HTTPS, and routes. Don't use `express`.
- `bun:sqlite` for SQLite. Don't use `better-sqlite3`.
- `Bun.redis` for Redis. Don't use `ioredis`.
- `Bun.sql` for Postgres. Don't use `pg` or `postgres.js`.
- `WebSocket` is built-in. Don't use `ws`.
- Prefer `Bun.file` over `node:fs`'s readFile/writeFile
- Bun.$`ls` instead of execa.

## Testing

Use `bun test` to run tests.

```ts#index.test.ts
import { test, expect } from "bun:test";

test("hello world", () => {
  expect(1).toBe(1);
});
```

## Frontend

Use HTML imports with `Bun.serve()`. Don't use `vite`. HTML imports fully support React, CSS, Tailwind.

Server:

```ts#index.ts
import index from "./index.html"

Bun.serve({
  routes: {
    "/": index,
    "/api/users/:id": {
      GET: (req) => {
        return new Response(JSON.stringify({ id: req.params.id }));
      },
    },
  },
  // optional websocket support
  websocket: {
    open: (ws) => {
      ws.send("Hello, world!");
    },
    message: (ws, message) => {
      ws.send(message);
    },
    close: (ws) => {
      // handle close
    }
  },
  development: {
    hmr: true,
    console: true,
  }
})
```

HTML files can import .tsx, .jsx or .js files directly and Bun's bundler will transpile & bundle automatically. `<link>` tags can point to stylesheets and Bun's CSS bundler will bundle.

```html#index.html
<html>
  <body>
    <h1>Hello, world!</h1>
    <script type="module" src="./frontend.tsx"></script>
  </body>
</html>
```

With the following `frontend.tsx`:

```tsx#frontend.tsx
import React from "react";
import { createRoot } from "react-dom/client";

// import .css files directly and it works
import './index.css';

const root = createRoot(document.body);

export default function Frontend() {
  return <h1>Hello, world!</h1>;
}

root.render(<Frontend />);
```

Then, run index.ts

```sh
bun --hot ./index.ts
```

For more information, read the Bun API docs in `node_modules/bun-types/docs/**.mdx`.

## Git workflow

- **PR-first.** Immediately after the user agrees on a scope/plan, the next action is `gh pr create --draft`. Order: branch → one initial commit → push → `gh pr create --draft` → then implement. The draft PR is the persistence anchor — without it, cron/loop-mode resumption has nothing to observe.
- The draft PR body uses this structure (so a resuming iteration knows where to pick up): `## In scope`, `## Explicitly out of scope`, `## Design rules applied`, `## Verifiability` (exact commands to run), `## Status` (Done / Remaining / Mark-ready criteria — keep this updated as work progresses via `gh pr edit`).
- Open one PR per feature. Don't bundle multiple features.
- Commit periodically — at milestones, a passing test, a working module, a completed sub-task. Don't pile up uncommitted work.
- `gh pr ready <num>` only after `bun typecheck && bun test` are green locally AND the manual verification command in the PR body succeeds.
- After marking ready in an interactive session, `gh pr view <num> --web` to open it in the user's browser for review. Skip in headless cron iterations.
- Never add `Co-Authored-By:` trailers to commits or PRs. Strip the default Claude Code co-author trailer before committing.

## Engineering rules — read every time

These rules override convenience, "best practice" muscle memory, and the urge to be thorough. Violate them and the codebase becomes the wrong thing.

### Crash over limp

When the program encounters state it doesn't expect — at any layer — **CRASH**. Do not try to recover. Do not log a warning and continue. Do not return `null` and let the next layer deal with it.

- Only catch errors you can name a specific recovery path for (`LLMRateLimit` → sleep + retry, `ToolFailure` → emit error tool_result for the model). Everything else propagates.
- Exhaustive `switch` with `assertNever` in the `default` branch.
- Invariant assertions at module entry — throw if precondition violated.
- A crash with a stack trace is infinitely cheaper to debug than a label committed with wrong data ten layers from the bug.

### No fallback values

Never invent a value to fill a missing/wrong-typed one.

- ❌ `const score = thing.score ?? 0`
- ❌ `const items = data.items ?? []`
- ❌ `const n = Number(input) || 0`
- ✅ `if (thing.score === undefined) throw new Error(...)` then `const score = thing.score`
- ✅ Schema-validate at boundaries (`zod.parse`), trust types internally
- ✅ Optional values get explicit `if (x === undefined)` branches — a real branch, not a silent substitute

### Single source of truth — never duplicate defaults

NEVER do this:
```ts
function foo(value = 20) { ... }     // default here
const value = 20                      // ALSO here

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theSalted/alphalabel](https://github.com/theSalted/alphalabel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
