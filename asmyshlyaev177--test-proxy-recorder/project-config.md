---
trigger: always_on
description: Record/replay HTTP+WebSocket proxy for deterministic e2e tests. pnpm monorepo.
---

# AGENTS — structure cheatsheet

Record/replay HTTP+WebSocket proxy for deterministic e2e tests. pnpm monorepo.

## Layout

```text
packages/
  test-proxy-recorder/   # the library + CLI (the product)
    src/
      ProxyServer.ts      # core: record / replay / transparent modes
      httpRecorder.ts     # capture & persist HTTP exchanges
      websocketHandlers.ts# WS record/replay
      replaySessions.ts   # per-test session keying (x-test-rcrd-id)
      cli.ts proxy-cli.ts proxy.ts  # `test-proxy-recorder` CLI entry
      init.ts             # `init` — auto-wire into a project
      config*.ts          # config load/merge
      reset.ts            # reset proxy mode
      nextjs/             # SSR helpers: registerProxyFetch / registerProxyAxios
                          #   / setNextProxyHeaders (middleware) / createHeadersWithRecordingId
      tanstack-start/     # TanStack Start SSR helpers: registerProxyFetch /
                          #   getRecordingId / createHeadersWithRecordingId
      recorderEnabled.ts  # shared isRecorderEnabled() gate for the adapters
      playwright/         # playwrightProxy fixture (before()/teardown())
      utils/              # redact, recordingId, cors, fileUtils, httpHelpers
    skills/               # AI agent skills (intent-managed; 500-line SKILL.md cap,
                          #   bulk in references/). Validate: npx intent validate <dir>
  landing/               # Astro Starlight docs site (the documentation site)
    src/content/docs/docs/  # English source of truth; other locales are translations
apps/                    # runnable examples + e2e suites (the proof the lib works)
  example-nextjs16, example-nextjs-edge, example-tanstack-start,
  example-websocket, example-auth-*, example-extension, example-init
```

Package exports: `.` (Playwright/core), `./playwright`, `./nextjs`, `./tanstack-start`. CLI bin: `test-proxy-recorder`.

## Commands (run from repo root)

```bash
pnpm build         # core library
pnpm lint
pnpm typecheck
pnpm test
pnpm example:test:e2e:ci                      # nextjs16 example — record then replay
pnpm example-edge:test:e2e:ci                 # edge example
pnpm example-tanstack:test:e2e:ci             # TanStack Start example
pnpm landing:dev
pnpm landing:build                            # docs site
pnpm landing:audits                           # accessibility + Lighthouse on the built docs site
```

## Site audits (docs site)

`pnpm landing:audits` → the token contract, then two Playwright projects driven by
`packages/landing/playwright.audits.config.ts`: `tests/a11y.spec.ts` and then
`tests/lighthouse.spec.ts`, which is held back by `dependencies: ['a11y']` so the
timing measurement has the box to itself. Everything lives in the landing package,
including the ~100 MB `lighthouse` dependency, because nothing else in the repo has
a use for it. `.github/workflows/lighthouse.yml` runs it on changes under
`packages/landing/**` and nowhere else. Both projects share one page list,
`tests/pages.ts`.

Both audit the **production build**, never `astro dev`: the `webServer` block runs
`pnpm run build && astro preview --port 4331`. Port 4331 is deliberately not 4321 —
both `astro dev` and `astro preview` default to that, and a dev server left running
would otherwise be silently accepted in place of the build.

### The accessibility gate

`tests/a11y.spec.ts` is one gate with two halves, both from
`@asmyshlyaev177/design-tokens` and both run against the same loaded page, in
**both themes** — Starlight's theme select defaults to `auto`, so the OS
preference is what picks the ramp on a first visit. Every assertion is
`expect.soft`, so an axe violation cannot hide a contrast failure beside it.

**axe**, at `COMPREHENSIVE_TAGS` (WCAG 2.0/2.1/2.2 A and AA, plus
`best-practice`). Its own contrast rules stay disabled by the package default —
the other half scores the same nodes on APCA as well as WCAG 2 — and
`incomplete` is asserted on, so text over a background image or an element below
the fold fails once and gets a decision rather than sitting unread.

**Rendered contrast** walks every visible text node and scores it against both
models. The floor is Lc 60, the weakest the token contract grants anything at
body size — not `--muted`'s 70. A DOM node does not say which token it used, so
a stricter floor fails sanctioned tokens.

Neither half is redundant with the Lighthouse accessibility score. Lighthouse
13.4 bundles axe-core 4.12 and runs 76 audits — 66 real rules plus 10 manual
checklist items that never execute — against axe's 104, in one theme only,
weighted into an average rather than a per-rule verdict. It scores no element
that holds no text node, which on a docs site is most of the chrome.

`integrations/expressive-code-a11y.mjs` is what the suite forced, and all three
of its parts close a gap the shipped defaults leave:

- **`liftThemeContrast`**, wired as expressive-code's `customizeTheme`. Night
  Owl leaves roughly a third of its tokens under the floor on Starlight's code
  background — comments worst, but keywords, punctuation and operators too. It
  raises each token's OKLCH lightness away from `theme.bg` until it clears both
  models, leaving hue and chroma alone.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [asmyshlyaev177/test-proxy-recorder](https://github.com/asmyshlyaev177/test-proxy-recorder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
