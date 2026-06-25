---
trigger: always_on
description: Open-source collaborative workspace (Notion-inspired). Next 16 (App Router) +
---

# open-silong — Claude / Agent Conventions

Open-source collaborative workspace (Notion-inspired). Next 16 (App Router) +
React 19 + Convex 1.36 (cloud reference deploy; self-hosted lane supported).
License MIT.

Live: https://silong-os.vercel.app (Vercel + Convex cloud prod
`canny-flamingo-7`, dev `coordinated-ptarmigan-140`) ·
Repo: https://github.com/rahmanef63/open-silong

**Self-hosted lane OFF 2026-06-04**: silong.rahmanef.com (Dokploy app
`notion-page-clone-gniidj`) stopped + autoDeploy disabled — push to main no
longer redeploys it. The Convex self-hosted compose
(`api-notion-page-clone.rahmanef.com`) still runs with the old data; restart
the app from Dokploy if the lane is ever revived. Vercel deployment is the
reference deploy now.

**Rebrand 2026-05-20**: project formerly known as `notion-page-clone` / `nosion`.
- Github repo renaming to `open-silong` (rahmanef63 account).
- Domain shifting `nosion.rahmanef.com` → `silong.rahmanef.com` (with redirect during transition).
- Convex backend domain shifting `api-silong.rahmanef.com` → `api-silong.rahmanef.com` (separate Dokploy ops).
- Internal code references to "Nosion" / "nosion" remain in many files; Phase 2 polish sweep will rebrand surface-by-surface. Backend `INSTANCE_NAME` stays `notion-page-clone` until a coordinated re-key (no urgency — internal id only).
- See `docs/rr-sync/2026-05-20-pivot-nosion-source-of-truth.md` for strategic rationale.

## Stack baseline

### Hard pins
- **Next ^16 + React ^19, Tailwind v4** — `proxy.ts` only, no `middleware.ts`.
  `experimental.cacheComponents` opt-in per page.
- **Convex self-hosted ^1.36** — Docker Compose on Dokploy node. Deploy via
  the pre-push hook (`scripts/install-pre-push.sh`) which sources
  `.env.local` then runs `pnpm exec convex deploy --yes`. Manual deploy:
  `set -a && source .env.local && set +a && pnpm exec convex deploy --yes`
  (raw `npx convex deploy` without env sourcing → `BadAdminKey`).
- **Auth = `@convex-dev/auth`** — NO Clerk. Custom auth slices only when
  documented insufficient.

### Vertical slices
- Layout: `frontend/slices/<slug>/` (UI + types) + optional
  `convex/features/<slug>/` (schema + queries + mutations).
- **Barrel-only cross-slice imports.** `@/features/<slug>` ✅ —
  `@/features/foo/lib/internal-thing` ❌. Barrels = contract.
- **Props-driven portability.** No hardcoded URLs / env names / role enums
  inside slice code — pass via props or env-configured allowlist.

### Convex non-negotiables
- Every client-reachable `mutation()` / `query()` declares
  `args: { v.* }` validators. Missing = P0.
- **No bare `.collect()`** — use `.withIndex(...).take(N)` or paginate.
- Server-side authz **inside the handler** — `requireOwned` /
  `requireWorkspaceMember` from `convex/_shared/`. Route gates don't
  protect HTTP queries.
- Indexes mandatory for every `.filter` / `.order` path. Add via
  `defineTable(...).index(...)`.

### UI non-negotiables
- shadcn primitives only. Never raw `<button>` / `<dialog>` /
  `<input type=date|file>`. Wrap via `ResponsiveDialog` / `DateField` /
  `FileUpload` from `frontend/shared/`.
- Theme tokens only (`bg-background` / `text-foreground` / `border-border`).
  No hex.
- Mobile-first responsive — `md:` / `lg:` layered up from single-column.
- `next/link` for internal routes, `next/image` for hosted assets. No raw
  `<a href="/internal">` / `<img>`.
- `NEXT_PUBLIC_*` = exposed in client bundle. Never secrets / admin emails /
  API keys.

### Delivery
- Solo dev → push direct to `main`, NO PRs. Conventional commits +
  `Co-Authored-By: Claude …` footer. Dokploy webhook auto-builds.
- Local CI: `/sc-git ci --repo notion-page-clone` or pre-push hook —
  no GitHub Actions cloud minutes.

### Slice metadata

Each slice ships a `slice.manifest.json` declaring its shared/slices/
convex dependency list. Regenerate via
`node scripts/generate-slice-manifests.mjs`. Portability blockers
(hardcoded routes / role enums / table-name leaks) are tracked by
`node scripts/audit-portability.mjs`.

### MCP

`convex/mcp/` is a Notion-canonical JSON HTTP surface.

### Before writing code
1. Check if the change crosses a rule above — apply even if user didn't
   mention it. Call out which rule when proposing.
2. New feature → check if it should be a new slice under
   `frontend/slices/<slug>/` + `convex/features/<slug>/`.
3. After editing: `pnpm typecheck` + relevant `pnpm test` before commit.
4. Found rule-violating existing code? Flag it, but only fix if user asks
   (avoid scope creep).

## Layout

- `app/` — App Router routes. Dashboard segments live under `/dashboard/*`.
  Pre-prefix legacy URLs (`/p/:id`, `/inbox`, `/settings`, …) redirect via
  `next.config.mjs` → `/dashboard/<same>`.
- `frontend/slices/{slug}/{components,views,hooks,lib}/` — feature slices.
  No `defineFeature()` / `config.ts` / `init.ts` shell here. Slices export
  via `index.ts` and are consumed directly by `app/` routes.
- `frontend/shared/{ui,lib,components,types}/` — primitives shared across
  slices.
  - `shared/components/icon-picker/` — DynamicIcon + IconPickerPopover
    (promoted from `slices/` 2026-05-11 because it was depended on by
    40+ files; lives in shared so it's available to consumers porting
    slices without dragging a peer slice).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rahmanef63/open-silong](https://github.com/rahmanef63/open-silong) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
