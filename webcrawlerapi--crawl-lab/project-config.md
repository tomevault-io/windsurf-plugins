---
trigger: always_on
description: - This file governs the entire repository until another AGENTS.md overrides it.
---

# Agents guide for Crawler Lab for LLM and coding agents
1. Scope
- This file governs the entire repository until another AGENTS.md overrides it.
- No nested AGENTS files currently exist; these rules apply everywhere.
- There are no Cursor rules (.cursor/ or .cursorrules) in this repo.
- There are no Copilot instructions (.github/copilot-instructions.md) in this repo.
2. Quick Facts
- Project name: crawl-lab (Scraper Tester API).
- Stack: Node.js (ES modules) with Hono.
- Entry: `src/index.js` creates the Hono app and exports default.
- Routing: configured in `src/routes.js` using handler modules.
- Config: `wrangler.toml` with `nodejs_compat` for Cloudflare Workers.
3. Dependency Management
- Use pnpm for installs (`pnpm install`); lockfile is pnpm-lock.yaml.
- Avoid npm/yarn unless explicitly requested.
- Keep dependencies minimal; prefer standard library and Hono utilities.
- Use exact version ranges already chosen unless a security fix is required.
4. Scripts and Commands
- Start (production-like, Node): `pnpm start` (runs `node src/index.js`).
- Dev watch (Node): `pnpm dev` (runs `node --watch src/index.js`).
- Cloudflare Workers: default entry is `src/index.js`; run via `pnpm dlx wrangler dev` or `pnpm dlx wrangler deploy` if needed (no script wired yet).
- Install deps: `pnpm install`.
- No build step beyond Node/Wrangler bundling; keep source ESM-compatible.
5. Tests and Linting
- There is no test suite configured; no `pnpm test` script exists.
- There is no lint/format command configured.
- If you add tests, prefer pnpm scripts (e.g., `pnpm test -- <pattern>` for single-test runs) and document them here.
- Until a linter is added, preserve existing style manually.
6. Repository Layout
- `src/index.js`: bootstraps app, conditionally starts Node server, exports default.
- `src/app.js`: builds Hono app and applies routes.
- `src/routes.js`: registers all endpoints and `notFound` handler.
- `src/handlers/*.js`: grouped endpoint handlers by domain (status, content, special, js, forum, size, assets).
- `src/pages/index.js`: builds the HTML index listing all routes.
- `public/`: static assets (`files/pdf/sample.pdf`, `images/sample.png`, `js/render.js`).
- `.wrangler/`: local wrangler state; avoid editing.
- `wrangler.toml`: worker config (compatibility date, flags).
7. Module and Import Style
- Use ES modules with explicit `.js` extensions for relative imports.
- Prefer `node:` prefix for built-in modules (e.g., `node:url`).
- Keep imports grouped: external packages first, then local modules.
- Use named exports per handler file (e.g., `export function handleX`); default export only for the app in `src/index.js`.
8. Formatting Conventions
- Indentation: 2 spaces; avoid tabs.
- Semicolons are used; keep them.
- Keep lines readable; favor wrapping long template strings via multiline literals.
- No automatic formatter is configured; maintain existing spacing and blank-line usage.
- Use trailing commas only where already present; do not reformat JSON-style blocks aggressively.
9. Strings and Templates
- Use template literals for multiline responses and HTML construction.
- Prefer single quotes for simple strings; use double quotes when matching JSON payload conventions.
- Keep embedded content at 200+ characters where endpoints expect substantial bodies for scraper testing.
- Escape backticks appropriately inside template literals.
10. Types and Data Handling
- Codebase is plain JavaScript; no TypeScript types are expected.
- Use explicit parsing for numbers (e.g., `Number.parseInt(..., 10)`), not implicit casting.
- Handle `undefined` safely when reading query params, env vars, or process globals.
- When reading request params/queries, default sensibly (`??`) and validate ranges.
11. Error and Input Handling
- Prefer early returns for invalid inputs with clear text responses (see `handleStatus`, `handleLongResponse`).
- Use Hono helpers (`c.text`, `c.html`, `c.json`, `c.newResponse`, `c.redirect`) instead of manual `Response` where possible.
- Set status codes explicitly in response helpers (second arg or options object).
- For empty bodies, use `c.newResponse(null, { status, headers })` and set `Content-Length: 0` when appropriate.
12. Routing Patterns
- Register routes in `setupRoutes` within `src/routes.js`; keep grouping by domain (status, js, special, content, forum, size, assets).
- Use path casing consistent with current endpoints (e.g., `/100Kb`, `/1Mb`, `/10Mb`).
- Default `notFound` handler returns descriptive text; keep consistency if extending.
13. Content Conventions
- Responses are intentionally verbose (200+ chars) to simulate real pages; maintain this when adding endpoints.
- HTML responses avoid styling/CSS to keep scraper behavior predictable.
- Content-type endpoints must set correct headers (`text/markdown`, `application/json`, `application/xml`, `text/html`, `text/plain`, `text/csv`, `text/tab-separated-values`).
- Forum endpoints support pagination via `?page=` query; preserve semantics if adjusting.
14. JavaScript-Rendered Pages
- JS-rendered endpoints are defined in `src/handlers/js.js` and rely on assets in `public/js/render.js`.
- Keep inline vs external distinctions: `/js/inline` embeds scripts, `/js/external` loads from `/js/render.js`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WebCrawlerAPI/crawl-lab](https://github.com/WebCrawlerAPI/crawl-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
