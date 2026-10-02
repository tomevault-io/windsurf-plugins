---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

**OpenFunnel** — an open-source, self-hostable alternative to Perspective.co /
Typeform. It renders JSON funnel documents into mobile-first, swipe-through quiz
funnels built for paid traffic, captures leads, and reports drop-off.

Bun workspace monorepo, AGPL-3.0-or-later, **zero runtime dependencies** (the
only `devDependencies` are `happy-dom` and `typescript`).

## Commands

```bash
bun install            # install workspaces
bun run dev            # runtime server with --watch on :3000  (apps/runtime)
bun run start          # runtime server, no watch
bun test               # full suite (engine + runtime smoke tests)
bun run typecheck      # tsc over the engine's JSDoc types + tsconfig.base.json
bun run demo           # zero-build static demo on :4321 (scripts/serve.mjs)

bun run scripts/check-no-deps.mjs         # no runtime deps in any workspace pkg
bun run scripts/check-engine-imports.mjs  # every engine import browser-resolvable
```

Run a single test file: `bun test packages/engine/test/logic.test.js`.

CI (`.github/workflows/ci.yml`) runs typecheck, the suite, and those two
invariant checks on every push and PR. The checks exist because both failures
they catch are invisible locally — Bun resolves a bare specifier and an
extensionless import happily, and the 404 only lands on a visitor's phone.

`bun test` (109 tests) and `bun run typecheck` both pass on `main` — keep it that
way. Two tests log expected warnings (`branch target "nope" not found`, an
invalid-URL `submitLead` failure); those are assertions about failure tolerance,
not breakage.

## Layout

```
packages/engine/     the funnel runtime — zero-dep browser ESM, no build step
apps/runtime/        the only backend — server.js (router) + lib/ + routes/
apps/app/            the console SPA (dashboard, builder, leads, analytics)
apps/builder/        legacy standalone builder UI  (superseded by apps/app)
apps/admin/          legacy standalone admin UI    (superseded by apps/app)
examples/*.json      funnel documents — this is the funnel "database"
demo/                offline zero-build demo page
.data/               JSONL lead/event sinks (gitignored)
```

### packages/engine

The engine mounts into any container in any framework and mutates nothing else.

- `src/types.js` — **JSDoc typedefs only, no runtime code.** This is the single
  source of truth for the funnel JSON contract. Change the contract here first.
- `src/controller.js` — the state machine. Owns `index`, `history`, `answers`,
  `lead`; mounts chrome; runs transitions; emits events; persists progress.
- `src/render/index.js` — builds the shared header, then dispatches on
  `step.type` to `choice` / `multiselect` / `form` / `content` / `loader` /
  `success`. `landing` is the one exception and returns *before* the header is
  built — see below.
- `src/render/landing.js` — the `landing` step: a full marketing page (hero,
  background media, nav, sticky CTA, sections, footer) whose CTAs advance into
  the quiz. It is what a cold ad click lands on, so a funnel is "landing page →
  questions → lead" in one document. Two rules it breaks on purpose:
  it owns the whole screen (no shared header — the hero draws its own eyebrow /
  headline / subtext, and drawing both prints the headline twice), and
  `step.blocks` is the page body *below* the hero rather than content above an
  interaction. Every section a page could want is therefore an ordinary
  `ContentBlock`, not a landing-only field, so adding one benefits every step
  type at once. The progress bar defaults to hidden on a landing step
  (`progress: true` overrides), and `width: "wide"` breaks the 9:16 phone frame
  on desktop via `data-width` on the root.
- `src/branching.js` — `resolveNext()`. Precedence: the interaction's `next` →
  the step's own `next` → linear fall-through. `null` ends the funnel.
- `src/piping.js` — `{{token}}` substitution from `lead` first, then `answers`.
  Unknown tokens render empty; a visitor must never see raw `{{...}}`.
- `src/analytics.js` — **ad-platform pixels only** (Meta, GA4/GTM, TikTok).
- `src/leads.js` — **your own backend only** (`/api/lead`, `/api/events`).
  Also captures UTM and click-id params (`gclid`, `fbclid`, `ttclid`, `ref`).
  These two files are separate on purpose; don't merge them.
- `src/theme.js` — funnel `theme` JSON → `--of-*` CSS custom properties, plus the
  eight `THEME_PRESETS` (`midnight-glass`, `neo-brutalist`, `warm-editorial`,
  `saas-gradient`, `clean-light`, `emerald-glow`, `violet-pulse`,
  `sunset-coral`). Also the only file in the engine that makes a third-party
  request: `loadThemeFont()` / `applyTheme(root, theme, { allowRemote })` fetch a
  non-system `theme.font` from Google Fonts. It stays independent of `consent.js`
  — the decision is passed in as a boolean.
- `src/consent.js` — the consent bar and the `marketingAllowed()` / `consentSignal()`
  gate for third-party sharing. Its header defines what is and is not gated.
- `src/persist.js` — localStorage resume. Fails silently by design.

### apps/runtime

`Bun.serve`, no framework. `server.js` is the router and nothing else (~175
lines): it owns the order routes are tried in and the admin gate, and delegates

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [luispdoesai/openFunnel](https://github.com/luispdoesai/openFunnel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
