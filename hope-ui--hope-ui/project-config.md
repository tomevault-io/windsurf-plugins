---
trigger: always_on
description: Operative index: the enforced rules, and where the rationale lives. Deep rationale is in
---

# CLAUDE.md

Operative index: the enforced rules, and where the rationale lives. Deep rationale is in
`__internal__/` (`plan.md`, `testing.md`, `solid-2.0-notes.md`, `definition-of-done.md`,
`theming.md`), plus a per-file usage doc per primitive under `__internal__/primitives/<src-path>/`.
End-user docs are separate: the website in `apps/docs/`.

## What this is

`hope-ui` — an elegant, themeable, accessible component library for **SolidJS 2.0 (beta)**, not 1.x.
Themeable components (Tailwind v4 + tailwind-variants) are the product, built over an **internal**
headless kernel (`@hope-ui/primitives`) that is an escape hatch, **not** a stability-promised API.
API-inspired by Base UI and React Aria — actively reference and adapt their code/reasoning.
Architecture: `__internal__/plan.md`. Build order: `__internal__/roadmap.md`.

**i18n provenance.** `@hope-ui/i18n`'s locale context is derived from React
Spectrum/`@react-aria/i18n` (Apache-2.0). **Do not rewrite it as a hand-rolled implementation** — the
deviations (SSR-safe seeding, `Symbol.for` dual-copy registry) are deliberate and documented in
`default-locale.ts`.

**`@solid-primitives` (`next` branch) is a dependency to adopt, not just reference.** Before writing
a new internal primitive, check it and record an *adopt / wrap / build-fresh-because* verdict
(`__internal__/solid-primitives-eval.md`). Anything adopted clears the full DoD through its consumer,
including the hydration round-trip. **Hazard:** an adopted dep creating a compute-form signal/memo
(`createSignal(fn)` / `createMemo`) must be **inlined** in the SSR harness — externalized, it resolves
a second `solid-js` copy and `_hk` diverges. `server.deps.inline` and the bridge's `ssr.noExternal`
both carry `/@solid-primitives\//`. Effect-only primitives are the safe bet.

**"SSR support" = "works in SolidStart"** — renders on the server, hydrates without mismatch, runs on
the client. Nothing broader. Verified with `renderToStream`/`hydrate` from `@solidjs/web`;
`@solidjs/start` is not on solid-js 2.0 yet, so that round-trip is the coverage. Four rules protect
it: effect-gate DOM access; `createUniqueId` for ARIA-linking ids; gate server-side `Portal` behind
`isServer`; keep an `aria-controls` IDREF only while its target is mounted. Details:
`__internal__/plan.md` § SSR & hydration requirements.

**Ships JSX-preserved source only**, under the `"solid"` export condition — the consumer's
`vite-plugin-solid` compiles per environment. No dom-compiled fallback: a consumer without that
plugin fails loudly. See `__internal__/plan.md` § Distribution model.

## Commands

```bash
pnpm install              # install workspace deps
pnpm build                # turbo: build all packages (tsdown → JSX-preserved .jsx + .d.ts per subpath)
pnpm lint                 # biome check .
pnpm format               # biome format --write .
pnpm typecheck            # turbo: tsc --noEmit per package (reads sibling src, never dist — see below)
pnpm test                 # vitest run --project=unit    (node, no DOM, pure logic)
pnpm test:ssr             # vitest run --project=ssr     (node, SERVER builds of solid-js + @solidjs/web)
pnpm test:browser         # vitest run --project=browser (real Chromium, DOM + hydration)
pnpm storybook            # visual harness on :6006 (the only non-test feedback loop)
pnpm build:storybook      # static build, also the CI smoke test for the Storybook config
pnpm check:coverage-parity  # DoD: per-file test+doc (primitives/theming); per-folder test+doc+story+ssr+hydration (components); no flat sprawl
pnpm check:class-forwarding # fails if a part's `get class()` drops the consumer's class (or omits it and never reads it)
pnpm check:recipe-purity  # fails if a preset recipe computes a color (color-mix / alpha modifier / magic opacity)
pnpm check:rtl-safety     # fails on a physical directional class (pl-/pr-/left-/text-right/…), CSSOM write or CSS declaration — packages + apps/docs
pnpm changeset            # NOT needed while the repo is at v0.0.0 — see "Changesets"
```

Playwright's browser, installed once (CI does this automatically):
```bash
pnpm exec playwright install --only-shell chromium
```

Single test file or test:
```bash
pnpm exec vitest run --project=browser packages/components/src/button/__tests__/button.browser.test.tsx
pnpm exec vitest run --project=browser -t "fires onClick"
```

Single package:
```bash
pnpm --filter @hope-ui/components build
pnpm --filter @hope-ui/components typecheck
```

## Git conventions

**Never add a `Co-Authored-By: Claude`, any `Co-authored-by`, or "Generated with Claude Code" trailer
to a commit message.** Commit messages carry the change rationale only.

## Third-party attribution

hope-ui is MIT. The references it ports from are not all MIT, so **adapting code and crediting a
reference are two different obligations** — `__internal__/reference-implementations.md` covers where to
look; this covers what you owe when you actually copy.

- **Designing against a reference's public API, ARIA pattern, or reasoning owes nothing.** Most files
  naming React Aria / Base UI / Angular Aria mid-body are in this bucket — leave them alone.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hope-ui/hope-ui](https://github.com/hope-ui/hope-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
