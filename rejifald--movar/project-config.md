---
trigger: always_on
description: > Movar is a cross-browser MV3 extension that keeps the internet in your language: it
---

# movar — agent guide (root)

> Movar is a cross-browser MV3 extension that keeps the internet in your language: it
> asks sites to serve your preferred language (Ukrainian-first, English fallback) and
> hides Russian content cards + Russian entries in on-site language pickers.

This file orients agents at the repo root. **Every workspace member has its own
`AGENTS.md`** with the detail you need to work in it without scanning — start there
once you know which member you're touching (links below).

## The mental model: two sequential layers

Movar's language handling is two layers that run in order on each page:

1. **Redirect layer** — ask the site to serve another language (URL params, picker
   click, hreflang, cookie/localStorage). Input: the picker's active marker, then
   markup/URL tiers (`<html lang>`, subdomain, path, self-hreflang). If it
   navigates, layer 2 is skipped.
2. **Content-filter layer** — atomically conceal individual cards whose detected
   language is blocked, behind a "curtain" overlay. Runs when layer 1 didn't navigate.

**Invariant:** the content layer never produces an aggregate verdict that feeds the
redirect layer (that would cause redirect/bounce "hiccups"). And Movar **blocks, never
translates** — translating Russian would launder it into trusted Ukrainian.

## Architecture: pure model packages vs. app orchestration

The content-script engine is split so the reusable logic is package-clean:

- **Pure model packages** — [`@movar/page-mode`](packages/page-mode/AGENTS.md),
  [`@movar/page-content`](packages/page-content/AGENTS.md),
  [`@movar/lang-pickers`](packages/lang-pickers/AGENTS.md),
  [`@movar/page-language`](packages/page-language/AGENTS.md). They **read** the DOM and
  build models but **never** import i18n, the curtain/tooltip overlays, or the
  page-mode color-scheme singleton. Consumed by both the extension and the diagnostics
  shadow-oracle.
- **App orchestration** — the DOM-mutating concealment (`content-conceal.ts` →
  `applyContentFilter`, `picker-filter.ts` → `filterPickers`), the overlays
  (`curtain.ts`/`tooltip.ts`), and the i18n catalog all live in
  [`apps/extension`](apps/extension/AGENTS.md). Don't move these into the model packages.

## Monorepo map

pnpm + nx workspace: `apps/*`, `packages/*`, `tooling/*`.

### Packages (libraries)

| Member                                                       | What it is                                                                                                       |
| ------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------- |
| [`packages/page-mode`](packages/page-mode/AGENTS.md)         | Page color-scheme (light/dark) detect / observe / apply (self-contained leaf)                                    |
| [`packages/page-content`](packages/page-content/AGENTS.md)   | Per-site content extractor **model** (DOM → `ContentNode` model); self-registering google/youtube extractors     |
| [`packages/lang-pickers`](packages/lang-pickers/AGENTS.md)   | On-site language-picker discovery / classify / active-lang / redirect-target **model**                           |
| [`packages/page-language`](packages/page-language/AGENTS.md) | Redirect-layer verdict: "what language is the site serving?" (consumes the picker model only)                    |
| [`packages/lang-detect`](packages/lang-detect/AGENTS.md)     | UA-vs-RU (+be/bg) text detection **and** BCP-47 code normalization (`normalizeBCP47`/`normalizeLanguageCode`)    |
| [`packages/host-match`](packages/host-match/AGENTS.md)       | Shared host predicates (`isGoogleHost`/`isYouTubeHost`) for the redirect + content-filter layers                 |
| [`packages/brand`](packages/brand/AGENTS.md)                 | Zero-dep brand constants leaf (`SUPPORT_EMAIL`, `FEEDBACK_URL`, `SOURCE_URL`)                                    |
| [`packages/settings`](packages/settings/AGENTS.md)           | Settings types/defaults + locked-language policy (`MovarSettings`, `defaultSettings`, `enforceLockedLanguages`)  |
| [`packages/events`](packages/events/AGENTS.md)               | Correction-event types (`CorrectionMechanism`, `CorrectionEvent`)                                                |
| [`packages/theme`](packages/theme/AGENTS.md)                 | Zero-dep design-token leaf — typed source of truth (colors/space/fonts/radii/breakpoints/sizes) → generated CSS  |
| [`packages/ui`](packages/ui/AGENTS.md)                       | React design-system primitives (+ pure `./tooltip-position`) consuming `@movar/theme`, for extension + marketing |

### Apps

| Member                                           | What it is                                                                                                                           |
| ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------ |
| [`apps/extension`](apps/extension/AGENTS.md)     | **The published product** — WXT MV3 extension (Chrome/Firefox/Safari): orchestration + concealment + overlays + i18n + popup/options |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rejifald/movar](https://github.com/rejifald/movar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
