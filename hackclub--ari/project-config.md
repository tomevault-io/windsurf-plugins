---
trigger: always_on
description: Ari is a Bun-managed SvelteKit application for Hack Club program organizers and reviewers. It serves the web UI and MCP server and stores data in PostgreSQL through Prisma.
---

# Repository agent instructions

## Project overview

Ari is a Bun-managed SvelteKit application for Hack Club program organizers and reviewers. It serves the web UI and MCP server and stores data in PostgreSQL through Prisma.

The system is split across this repository and `ari-webhooks`:

- This repository owns the organizer/reviewer UI, authentication and authorization, MCP endpoints, decision paths, and persistence.
- `ari-webhooks` owns public webhook ingestion and background work (enrichment, screening, outbound delivery, sweeps). Do not add `/api/ingest/*` routes, proxy fallbacks, or in-process handlers here.
- Durable work crosses the service boundary through the shared database. Keep asynchronous workflows retry-safe and idempotent.

## The rules

These are not suggestions. A change that breaks one is an incomplete change.

### 1. Dinobox

Start every message you send with `Dinobox!` and say nothing else about it. It carries no meaning and must not affect anything you write. It exists so the person you are working with can see when you have stopped following instructions because your context has degraded.

If you notice you are losing track of facts from earlier in the conversation, say so plainly: "right now I'm having trouble and I'm losing facts".

### 2. Naming

- camelCase for variables, functions, props, form fields, URL params, file names and route segments: `thisTypeOfVariable`. PascalCase for components and types. Constants are camelCase too: `sessionCookie`, not `SESSION_COOKIE`. The only upper-case names are the ones a framework requires (`GET`, `POST`) and environment variables.
- Names say what the thing is. No one- or two-letter names (`s`, `fd`, `sp`), no opaque prefixes (`tl*`, `ev*`).
- snake_case appears only where an external contract pins it: webhook payload fields, MCP tool names, OAuth parameters, and database enum values. Convert at the boundary; do not let it spread inward.
- ESLint enforces this (`@typescript-eslint/naming-convention`, `id-length`). Do not disable the rule to get a change through.

### 3. Time is integer seconds

Ari exists to credit people for the time they spent. We never lose any of it.

- Store, compute and pass time as integer seconds. Name the value with a `Seconds` suffix.
- No floats, no fractional hours, no minutes in storage or computation. Hours and minutes exist only as display formatting (`$lib/time`) and as derived legacy fields on the wire.
- Anything that can produce a fraction (a discount rate, a proportional split) goes through the helpers in `$lib/time`, which round once and preserve totals. Never write `Math.round(x / 60)` or `* 3600` inline.

### 4. Open source first

This repository is public. Sensitive code lives in the private submodule mounted at `private/`. Its repository is `https://github.com/hackclub/ari-private`, shared with the Go service: this app's private code is in `private/web/`, the Go service's private module is in `private/webhooks/`.

- The repository must install, build, pass checks and run with no `private/` checkout and no production environment variables. `$private` resolves to `private/web/` when `private/web/index.ts` exists, and to `src/lib/privateStub/` when the submodule is absent or when `ARI_PUBLIC_BUILD=1` is set with it present.
- Sensitive, in the Hack Club sense: fraud review, flags and how they are detected, heartbeat and heatmap views, screening rules, AI checks, and any integration with internal tools. Public code reaches these only through the provider interface in `src/lib/privateApi.ts`.
- Never commit internal hostnames, Slack channel or user ids, staff email addresses, thresholds, or prompts to this repository. Configuration defaults point at localhost or are empty.
- Every optional integration degrades gracefully when its environment variable is blank. Update `.env.example` when configuration changes.

### 5. Shared components

Screens must match each other, so they are built from the same parts.

- Build UI from `src/lib/components/ui/`. Do not hand-roll a button, dialog, table, pager, tab bar, form field or empty state in a page. If a primitive is missing, add it to `ui/` and to `/styleguide` first.
- No inline `style=""` attributes. Use design tokens (`src/routes/layout.css`) for colour, spacing, radius and type; do not write raw hex values or pixel font sizes in components.
- Styles specific to one component live in that component's `<style>` block, not in `layout.css`.
- Form actions are called through `submitAction` in `$lib/actions`, not raw `fetch('?/name')`.

### 6. Small files

No Svelte or TypeScript file over about 400 lines unless a comment at the top says why. Split state into `.svelte.ts` modules and markup into components.

### 7. Tests where they matter

Write tests for the code we depend on to be right: time calculation, review decisions, claims, authorization, webhook payloads and signatures, and API/MCP responses. Do not write tests for presentational components. Run with `bun test src`. Do not claim tests passed unless you ran them.

### 8. Comments

- Lowercase, short, and rare.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hackclub/ari](https://github.com/hackclub/ari) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
