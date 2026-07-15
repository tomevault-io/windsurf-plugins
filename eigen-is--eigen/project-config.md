---
trigger: always_on
description: Handles file CRUD, path resolution, storage key building. Three storage backends: `local` (hierarchical
---

# AGENTS.md — Eigen Project Context

Eigen is a self-hosted Google Workspace alternative. Monorepo with integrated apps sharing a single API server, UI
library, and business logic layer.

## Tech Stack

- **Runtime**: Bun (server + client)
- **Backend**: Elysia + Drizzle ORM (SQLite)
- **Frontend**: React 19 + TypeScript + TanStack Router + TanStack Query + Eden Treaty (type-safe API client)
- **UI**: Tailwind CSS 4 + shadcn/ui + Lucide React
- **Auth**: better-auth (email/password, 2FA, orgs, teams)
- **Real-time**: Yjs (collab editing), WebSocket, SSE (notifications)

## Project Structure

```
apps/
  api/          # Elysia backend (port 8000)
  mail/         # Email client
  drive/        # File storage
  docs/         # Document editor (Yjs/Tiptap)
  contacts/     # Contact management
  calendar/     # Calendar + scheduling
  chat/         # Real-time chat (MUD-inspired)
  stickies/     # Kanban board (Yjs)
  slides/       # Presentations (Yjs)
  sheets/       # Spreadsheets (sheet engine + Yjs)
  space/        # Team workspace
  admin/        # Org/team admin + first-run setup wizard
  index/        # Landing page

packages/
  lib/          # @workspace/lib — shared types, hooks, API client, SSE handlers, validation
  ui/           # @workspace/ui — shared shadcn components, layout system
  sheet/         # Spreadsheet engine + UI (forked from fortune-sheet/luckysheet)

data/           # Runtime storage (databases, user files)
docs/           # Architecture documentation
```

## Development

```bash
bun run serve          # All apps + API
bun serve:mail         # Single app + API
bun run lint           # Lint + format check (biome)
bun run lint:fix       # Auto-fix lint + format issues
bun run typecheck      # Type check all packages
bun run test           # API integration tests
bun run check          # lint + typecheck + test
```

### Critical Rules

- **No AI co-author trailers in commits** — never add `Co-authored-by: Claude/Copilot/...` lines to commit
  messages, even if your tooling defaults to it
- **Read [CODE-STANDARDS.md](docs/CODE-STANDARDS.md) before writing code** — defines typing rules, code style,
  common LLM mistakes with BAD/GOOD examples, and the **self-review checklist**. Must be followed before declaring
  any task complete
- **Read existing code before writing new code** — read 2-3 existing files in the same directory. Match their style,
  structure, naming, and patterns exactly. New code must look like it was always there
- **Search [SHARED-PRIMITIVES.md](docs/SHARED-PRIMITIVES.md) before building a shared hook, component, type, or
  util** — the generated index of everything `packages/ui` + `packages/lib` export. Import what already exists;
  if it's missing, export it from the package barrel so it gets catalogued. `bun run primitives` regenerates it
- **Read relevant docs before planning or coding** — check `docs/` for architecture docs on the domain you're
  touching (e.g., `docs/COMMENTS.md` before adding comment features, `docs/EXPORT.md` before changing export).
  Don't assume you know the conventions — verify them
- **Always run `bun run check`** after changes (lint + typecheck + test). When multiple agents run in parallel,
  only the main agent should run check — concurrent runs cause deadlocks
- **Code goes in the right layer** — hooks/mutations in `packages/lib/src/core/[domain]/hooks/`, shared types in
  `packages/lib/src/types/`, shared UI in `packages/ui/`, app-specific code in `apps/`. Rule of thumb: if two or
  more apps need it, it belongs in `packages/`. Never put `useQuery`, `useMutation`, error toasts, or `try/catch` +
  `toast.error()` in app components — all error handling lives in hooks using `onMutationError`.
  See [NOTIFICATIONS.md](docs/NOTIFICATIONS.md)
- **Package dependency direction is one-way: `sheet → lib`, never the reverse.** `packages/lib` is shared
  FE+BE; `packages/sheet` has React peer dependencies and DOM-coupled modules. If lib imported sheet,
  the BE would transitively pull React in at module-eval time. Shared sheet types (`Cell`, `Sheet`, `Op`,
  `CellMatrix`, `Range`, `SingleRange`, `ConditionalFormatRule`, …) live in `packages/lib/src/sheets/types.ts`;
  the sheet package's `engine/types.ts` and `state/types.ts` re-export them. Sheet utilities that need to be importable
  by both FE and BE (e.g. `opToPatchOnSheets`) live in `packages/lib/src/sheets/`
- **Don't break the type chain** — types flow from Elysia route handlers → Eden Treaty → hooks → components
  automatically. No `as any`, no `as Type` casts. Fix types at the source (add return type annotations to backend
  handlers using shared types from `packages/lib/src/types/`). See CODE-STANDARDS.md § Typing
- **Backend errors use `ApiError`** — `throw new ApiError(status, message)` for HTTP errors.
  `throw new Error()` only for internal invariants (db not open, missing config)
- **Think about every `await`** — a bare async call returns a truthy Promise (dangerous in conditionals: `if
  (!asyncFn())` is always false). Fire-and-forget must have `.catch()`. Skip `await` when blocking would hurt
  response time and failure is acceptable

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eigen-is/eigen](https://github.com/eigen-is/eigen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
