---
trigger: always_on
description: Default to using Bun instead of Node.js.
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

## Dev servers & builds — ASK FIRST

Do NOT start new dev servers (vite, `bun run dev`, `bun --hot`, etc.) and do NOT run builds/deploys on your own initiative. The user always has dev servers running on specific ports. Ask the user which server/port the dev environment is running on, then use it (e.g. for browser preview). If you think a build or deploy is needed, ask first — the user knows the setup.

## Sepia docs — keep in sync when adding features

Sepia's docs are the agents' only window into the server. When you add a feature (new tool, action, field, endpoint, or behavior change), update ALL of these before calling it done:

1. `src/instructions.ts` (MEMORY_CONTRACT) — the MCP `instructions` field, injected fresh every session. THE source of truth.
2. `skills/sepia/SKILL.md` — the on-demand agent skill.
3. `skills/sepia/always-on/*` (vscode, claude, cursor, agents, opencode, zed) — always-on essentials. Keep them minimal + stable: new depth goes in llms.txt / SKILL.md, not here.
4. `llms.txt` — full capability overview, served at /llms.txt. What agents fetch when they need the whole surface.
5. `scripts/gen-skill-ref.ts` — run `bun run scripts/gen-skill-ref.ts` to regenerate `skills/sepia/references/tools.md` from the schemas.
6. `scripts/smoke.ts` — cover the new surface (a feature without a smoke check is untested).
7. Version — bump `DOCS_VERSION` in `packages/shared/src/types.ts` and run `bun run scripts/stamp-docs-version.ts` (stamps the shared `sepia-docs-version` marker into always-on/*, llms.txt, AGENTS.md). NOTE: the `version:` frontmatter keys in SKILL.md / vscode / cursor are those files' OWN versions (managed by the skill system) — the stamp script never touches them.
8. Installed copies — run `bash scripts/install-skill.sh` (block-marker sections update in place, never duplicate).
9. Deploy — `fly deploy` so the served versions (/version, /llms.txt, /skill, /instructions/\*, MCP instructions) update for everyone.
10. Check — `bun run scripts/check-docs-version.ts` after deploy to confirm installed copies are current.

Rule: no feature is done until its docs are updated. Check the diff of every always-on file before committing.

<!-- sepia:start -->

## Sepia memory (always-on) — AGENTS.md (Codex / OpenCode / generic)

<!-- sepia-docs-version: 1.0.0 -->

You are connected to the user's personal Sepia memory server (sepia) over MCP (any `AGENTS.md`-aware agent: Codex, OpenCode, Copilot, Cursor, Zed). It stores a knowledge graph in namespaces (default `personal`): entities, relations, memories with importance scoring.

> **MANDATORY:** This file is auto-loaded in every session by all `AGENTS.md` clients. Your first tool call on any meaningful task MUST be `search`. If you answer without searching, you are guessing.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Michael-Obele/sepia](https://github.com/Michael-Obele/sepia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
