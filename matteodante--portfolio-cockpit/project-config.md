---
trigger: always_on
description: Notes for Claude Code in this repo.
---

# CLAUDE.md

Notes for Claude Code in this repo.

## Stack

Next.js 16 (App Router), React 19.2 + React Compiler, TypeScript strict,
Three.js 0.183 vanilla (NOT R3F), Tailwind v4, Zustand, OpenAI Responses
API. Bun 1.3 (pkg manager + test runner). Biome 2.4. `tsgo` for fast
typechecks. Node 22. EN/IT.

## Commands

```bash
bun run dev              # setup:styles + next dev (Turbopack)
bun run build            # setup:styles + next build
bun run check            # biome + tsgo + bun test (CI-equivalent)
bun run lint:fix         # biome lint --write --unsafe
bun run typecheck        # tsgo --noEmit
bun run typecheck:tsc    # fallback to stock tsc
bun run setup:styles     # regen tailwind.css + root.css from TS config
bun run encrypt:private  # re-encrypt private-src/ → .enc blobs in repo
bun run analyze          # ANALYZE=true bun run build
```

Single test: `bun test path/to/file.test.ts -t "name"`. Bun runs tests
directly. No Jest, no Vitest layer.

`setup:styles` runs before every dev/build and overwrites
`lib/styles/css/{tailwind,root}.css`. Banner says do not edit. Edit
`lib/styles/config.ts` (and re-exported modules: `colors.ts`,
`typography.ts`, `easings.ts`, `layout.mjs`).

## What this is

One page route. One Three.js scene. Chat + unlock + gated CV /
translations APIs. No CMS, no DB, no marketing pages.

## Routing

- `app/[lang]/page.tsx` → `CockpitLauncher` → dynamic-imports
  `CockpitApp` with `ssr: false`. Scene is client-only.
- `proxy.ts` (root) — Next 16 middleware. Detects locale from
  `Accept-Language` via `@formatjs/intl-localematcher` + `negotiator`
  and redirects `/foo` → `/{locale}/foo`. Matcher excludes `_next`,
  `api`, paths with extensions.
- `lib/i18n/config.ts` — `'en' | 'it'`. Invalid locale → `notFound()`.
- `lib/i18n/index.tsx` — flat-key React context. `useT()` falls back
  to the key. Dotted keys: `cockpit.sections.about.label`.

## Cockpit

`components/cockpit/cockpit-app.tsx` owns two state vars:

- `near` — planet within docking range
- `docked` — section in overlay (or `null`)

Composition (single `position: fixed` container):

- `scene/cockpit-scene.tsx` — ~1150 lines of imperative Three.js. Owns
  its RAF loop, physics constants, post-processing (`EffectComposer`
  + `UnrealBloomPass`, custom god rays). Pushes to React via
  `onNearChange` / `onDockRequest` callbacks AND via the Zustand HUD
  store (`setHud`).
- `chrome/*` — non-interactive HUD panels (top bar, side consoles,
  bottom console, frame). Read from `useHud`.
- `dock/dock-overlay.tsx` — full-screen overlay. Section bodies in
  `dock/sections/*`.
- `COMM_SECTION` — dock-only. Not a planet. Reached via the
  bottom-console COMM button.

`Escape` undocks. Owned by `cockpit-app.tsx`, not the scene.

## State

- React state (`near`, `docked`) lives in `cockpit-app.tsx`.
- `lib/hooks/cockpit-store.ts` — `useHud` (Zustand) for HUD gauges
  (`speed`, `coords`, `gravity`, `landed`, `phase`, `nearestId`).
  `setHud` diffs before `setState` so the scene can call it every
  frame without renders. Use `setHud(patch)`, not `useHud.setState`.

## Sections data

`lib/data/cockpit-sections.ts` — single source of truth for planet
layout (position, radius, color, emissive, ring/earth flags) and the
`CockpitSectionId` union.

To add a planet:

1. Add an entry to `SECTIONS`.
2. Add `cockpit.sections.<id>.*` keys to both translation files.
3. Add a case in `dock/dock-overlay.tsx`.

## Chat API

`app/api/chat/route.ts` is the only chat backend.

- `runtime = 'nodejs'`. No `force-dynamic` — cookie reads already
  opt the route into dynamic rendering.
- Zod caps: 20 messages, 500 chars each, 4000 chars total.
- Rate limit via `lib/api/rate-limit.ts` (`createRateLimiter`):
  chat 10/60s/IP, unlock 5/60s/IP. Uses Upstash Redis when
  `UPSTASH_REDIS_REST_URL` / `UPSTASH_REDIS_REST_TOKEN` are set,
  in-memory sliding window otherwise. `getClientIp` reads
  `x-vercel-forwarded-for` (signed by Vercel edge). On Vercel, an
  edge Firewall rule adds another layer.
- Moderates the latest user message via `omni-moderation-latest`.
  Fails open if moderation itself errors.
- `openai.responses.create({ stream: true })` → pipes
  `response.output_text.delta` into a `ReadableStream<Uint8Array>` as
  plain text. No SSE framing.
- `INSTRUCTIONS_HEAD` + `PROFILE_PUBLIC` + `INSTRUCTIONS_TAIL` is the
  public system prompt. When `hasAccess()` (cv_access cookie) returns
  true, the private profile is loaded from `lib/ai/chat-profile.enc`
  (AES-GCM, lazy-cached) and replaces `PROFILE_PUBLIC`.
  `{{LOCALE_LANGUAGE}}` is templated per request.
- Model `gpt-5.4-nano` is real and intentional. Do not "fix" it to
  `gpt-5-nano` or any variant. Confirmed by the project owner.

Client: `dock/sections/comm-chat.tsx` reads the stream and renders
markdown via `react-markdown`.

## CV access gate

Detailed CV, private translations, and the full chat profile are
gated behind a shared access code. `/llms.txt` is always public.

- `POST /api/unlock` validates a password (`CV_ACCESS_PASSWORD` env)
  and sets an HMAC-signed cookie `cv_access` (`CV_ACCESS_SECRET` env,
  30 days, httpOnly, secure in prod, sameSite=lax).
- `DELETE /api/unlock` clears the cookie.
- `lib/auth/cv-access.ts` — `hasAccess()` reads cookie via Next's

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matteodante/portfolio-cockpit](https://github.com/matteodante/portfolio-cockpit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
