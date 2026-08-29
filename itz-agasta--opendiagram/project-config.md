---
trigger: always_on
description: OpenDiagram - AI diagram generator for software architecture. Describe your system in plain English, get editable diagrams on an Excalidraw canvas.
---

# AGENTS.md

OpenDiagram - AI diagram generator for software architecture. Describe your system in plain English, get editable diagrams on an Excalidraw canvas.

Bun 1.3 monorepo. `apps/web` Next.js 16 (:3001), `apps/server` Hono (:3000), `apps/fumadocs` (:4000), `packages/{auth,config,db,env,harness}`.

## Commands

```bash
just web               # For injecting infisical dev secrets
just server            # If you wnat fast + .env based secret rotate use below bun run cmds:
bun run dev:web        # one process each, both backgrounded, started separately:
bun run dev:server     # the turbo TUI (`bun run dev`) segfaults on this machine
just check             # oxlint + oxfmt --write
just types             # tsgo across the workspace
just db-generate <name>  # migration from schema changes
just db-migrate          # apply pending migrations
just db-seed             # plan limits from packages/db/src/seed.ts (--dry-run to preview)
just db-setup            # migrate + seed, in that order; a fresh DB needs both
just clean | just reinstall
```

Run `just check` and `just types` before calling a coding session done.

## Repo conventions

- **Next.js 16 is not the Next.js you know.** Breaking changes to APIs, conventions, and file structure. Read `node_modules/next/dist/docs/` before writing app-router or config code, and heed deprecation notices.
- Install with `bun add`, never by hand-editing package.json. Workspace deps are `workspace:*`. `catalog:` is only for deps used by **two or more** packages.
- `@/` aliases `apps/web/src/`. Shared components live in `components/`, page-specific ones in `components/<feature>/`.
- **`apps/server` and `packages/*` files stay under 300 LOC, comments included.** Past that, split. Give the pieces a real structure - a directory with a narrow entry point, the way `lib/quota/` and `lib/dodo/` already do - rather than cutting wherever line 300 lands. Does not apply to `apps/web`, where vendored shadcn components skew the count.
- Typed env: import from `@OpenDiagram/env/web` or `@OpenDiagram/env/server`.
- `packages/db`: never acquire nested DB connections.
- Interactive controls must look interactive: `cursor: pointer` from the global stylesheet. Only override for disabled/loading (`cursor-wait`, `cursor-not-allowed`).
- No em dashes and no `--`. Prose, comments, commit messages.
- Never guess an API. context7 MCP for known libraries, Exa for obscure packages / platform APIs / specific URLs, ask the user if neither settles it.

## Harness (packages/harness) - read before touching diagram code

The diagram engine. Full docs: `packages/harness/README.md`. Non-negotiables:

- **LLM never chooses pixels/colors/fonts.** It emits a semantic `DiagramSpec`; layout (ELK / sequence grid) + themed renderer own all geometry and styling. Don't add visual fields to the spec.
- **Sizing and rendering must agree:** `measure.ts#nodeSize` reserves the box the renderer draws into. Change both branches together.
- **Edge routes are drawn verbatim.** Labels are measured against ELK's exact polyline; never reroute after layout. Excalidraw `elbowed` arrows don't work via programmatic insert.
- **No `@excalidraw/excalidraw` imports inside the harness** (browser-only package). Skeleton to element conversion lives in `apps/web/src/lib/excalidraw-utils.ts`, which must pass fresh elements through `restoreElements` (paint-skip bug otherwise).
- **`bun --hot` does NOT reload harness edits.** Restart `dev:server` or you verify stale code.
- **Zod spec schema stays Gemini-safe:** no `.refine()/.default()/.transform()`. Gemini reliably typos `from1` for `from` in edges. `experimental_repairToolCall` in `routes/diagram.ts` fixes it deterministically; don't remove it.
- Measured negative result: `elk.layered.nodePlacement.strategy: NETWORK_SIMPLEX` makes routing worse. Don't re-add. See `future.md` for the roadmap.
- **After ANY harness change run `bun test` in `packages/harness`** (`test/harness.test.ts` - geometry smoke suite: sequence fragments, ERD crow-feet, orthogonal routes, column alignment). Extend it when you add pipeline features.

## Verifying a change against the running app

**`apps/server/.env` `DATABASE_URL` points at PRODUCTION.** Test fixtures are real rows. Clean them up, and never run destructive SQL without saying so first. (Temporary: the prod DB and its env get wiped before launch.)

`next-server` exiting **143 is earlyoom**, not your bug.

Server Sentry needs `--preload @sentry/node/preload` (dev script, start script, Dockerfile `CMD`). The Hono middleware inits after `pg` is already imported, so without the flag every `db` span silently vanishes. Same reason `bun build --compile` cannot be used.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Itz-Agasta/OpenDiagram](https://github.com/Itz-Agasta/OpenDiagram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
