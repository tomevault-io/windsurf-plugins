---
trigger: always_on
description: <cloudflare-workers-monorepo>
---

<cloudflare-workers-monorepo>

<title>Cloudflare Workers Monorepo Guidelines for Claude Code</title>

<commands>
- `just install` - Install dependencies
- `just dev` - Run development servers (uses `bun runx dev` - context-aware)
- `just test` - Run tests with vitest (uses `bun vitest`)
- `just build` - Build all workers (uses `bun turbo build`)
- `just check` - Check code quality - deps, lint, types, format (uses `bun runx check`)
- `just fix` - Fix code issues - deps, lint, format, workers-types (uses `bun runx fix`)
- `just deploy` - Deploy all workers (uses `bun turbo deploy`)
- `just preview` - Run Workers in preview mode
- `just new-worker` (alias: `just gen`) - Create a new Cloudflare Worker
- `just new-package` - Create a new shared package
- `just update deps` (alias: `just up deps`) - Update dependencies across the monorepo
- `just update pnpm` - Update pnpm version
- `just update turbo` - Update turbo version
- `bun turbo -F worker-name dev` - Start specific worker
- `bun turbo -F worker-name test` - Test specific worker
- `bun turbo -F worker-name deploy` - Deploy specific worker
- `bun vitest path/to/test.test.ts` - Run a single test file
- `pnpm -F @repo/package-name add dependency` - Add dependency to specific package
</commands>

<architecture>
- Cloudflare Workers monorepo using pnpm workspaces and Turborepo
- `apps/` - Individual Cloudflare Worker applications
- `packages/` - Shared libraries and configurations
  - `@repo/oxlint-config` - Shared oxlint configuration
  - `@repo/typescript-config` - Shared TypeScript configuration
  - `@repo/hono-helpers` - Hono framework utilities
  - `@repo/tools` - Development tools and scripts
- Worker apps delegate scripts to `@repo/tools` for consistency
- Hono web framework with helpers in `@repo/hono-helpers`
- Vitest with `@cloudflare/vitest-pool-workers` for testing
- Syncpack ensures dependency version consistency
- Turborepo enables parallel task execution and caching
- Workers configured via `wrangler.jsonc` with environment variables
- Each worker has `context.ts` for typed environment bindings
- Integration tests in `src/test/integration/`
- Workers use `nodejs_compat` compatibility flag
- GitHub Actions deploy automatically on merge to main
- Changesets manage versions and changelogs
</architecture>

<code-style>
- Use tabs for indentation, spaces for alignment
- Type imports use `import type`
- Workspace imports use `@repo/` prefix
- Import order: Built-ins → Third-party → `@repo/` → Relative
- Prefix unused variables with `_`
- Prefer `const` over `let`
- Use `array-simple` notation
- Explicit function return types are optional
</code-style>

<client-contract-notes>
Response shapes the Rec Room client depends on. These were found by watching the live
client, not by reading a spec: when one is wrong the client renders nothing or hangs
rather than erroring, so tests won't catch a regression. Don't "clean up" an
inconsistency here without checking the client first.

- Player image lists (`api`: `/api/images/v5|v4/player/:id`, `/api/images/v3/feed/player/:id`)
  must use the `toImagesPlayer` projection — `Id` → `SavedImageId`, `Type` →
  `SavedImageType`, no `TaggedPlayerIds`. Serving the raw `SavedImage` renders blank
  thumbnails.
- The room photo feed (`api`: `/api/images/v4/room/:roomId`) serves the raw `SavedImage`
  and displays correctly. It is deliberately NOT projected — do not unify these two.
- A club's `AdditionalImages` (`clubs`) is an array of whole `SavedImage` records, not
  image names — a bare string array fails the client's parser ("expected '{'"). The list
  is packed: removing an image shifts the rest up, never leaving a blank slot.
- A room's `LoadScreens` (`rooms`: `PUT /rooms/:id/loadscreen`) is an array — the
  client's parser wants one — but the client renders only the FIRST entry and only ever
  posts one. So the endpoint REPLACES the list rather than appending: an appended screen
  sits unreachable behind the old one and setting a load screen looks like it did
  nothing. Keep the array shape for eventual multi-screen support.
- Endpoints the client re-renders from must return the updated entity, not
  `{ error, success, value: null }` — e.g. `clubs` `PUT /club/:id/clubhouse` left the old
  clubhouse on screen until it answered the full details envelope.
- Every subroom mutation (`rooms`: create, delete, `/subrooms/:sid/clone`,
  `/subrooms/:sid/accessibility`, `/subrooms/:sid/publish_save`) answers
  `{ success, error, value }` with the whole updated ROOM — the client re-renders the room
  from `value`. Notably `value` is the room even for `clone`, whose product is a new
  SUBROOM; only the room-level `POST /rooms/:id/clone` returns the thing it created.
- The room save (`rooms`: `POST /subrooms/:sid/data`) is the ONE exception to that shape:
  `value` is `{ room, subRoomDataSave }`, and `error` is NULL rather than `""`. The
  `subRoomDataSave` is camelCase with a different field set from the PascalCase
  `CurrentSave` embedded in the room (no persistence/OM/UGC versions, no moderation state,
  no asset arrays; but `unityAsset`/`unityAssetHash`). Don't unify the two projections.
- A subroom's saved scene loads from `CurrentSave.DataBlob` (`rooms`: `GET /rooms/:id`),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [recflare/server](https://github.com/recflare/server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
