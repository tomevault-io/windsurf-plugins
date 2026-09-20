---
trigger: always_on
description: This file is the always-loaded rulebook for any AI coding agent working in
---

# Agent guardrails for Pinkless

This file is the always-loaded rulebook for any AI coding agent working in
this repo (Codex, etc.). Claude Code users get the same rules as
auto-triggering skills in `.claude/skills/` — this file exists so agents that
don't read that format (Codex and others) still get them. Ground truth for
all of it is `Files/REQUIREMENTS.md`; when this file and REQUIREMENTS.md
disagree, REQUIREMENTS.md wins — re-read it if something here looks stale,
since it changes as the team builds.

Product in one line: on a Kroger product page for an item marketed to women,
Pinkless shows a reviewed men's or neutral equivalent that costs less **at the
same Kroger store**, with both prices from Kroger's official API. Kroger is the
only retailer (the CVS/Walmart cross-retailer design is superseded).

Architecture in one line: `apps/extension` extracts product identity on
kroger.com pages and sends only that identity + selected Kroger store to
`apps/api` (Vercel serverless), which holds the Kroger credentials, prices the
current product and its reviewed equivalents through the Kroger provider, and
returns a comparison; `packages/matcher` does eligibility/savings logic;
`packages/catalog` holds products (`products.json`) and reviewed
women's→men's/neutral pairs (`equivalences.json`); `apps/marketplace` calls the
same read-only API (`GET /api/comparisons`).

---

## 1. Frontend craft (`apps/marketplace`, extension popup/badge UI)

- Design tokens live in `packages/tokens/` (`tokens.json` is source of truth,
  `tokens.css` is compiled). See `DESIGN_SYSTEM.md`.
- Always style with the CSS variables (`var(--surface-pink)`, `var(--ink)`,
  `var(--space-4)`, `var(--radius-md)`, etc.) and the type classes
  (`.display`, `.heading`, `.body`, `.caption`, `.label`). **Never** write a
  raw hex value or a magic font-size/spacing number. If a needed value isn't
  a token yet, stop and ask — don't invent one (there's an open question
  about primary/CTA color; don't silently resolve it).
- The extension badge renders inside a Shadow DOM and cannot inherit
  page-level CSS. Never link an external stylesheet into the shadow root —
  use `mountBadgeRoot()` from `apps/extension/src/content/shadow-root.ts`,
  which injects the generated `TOKENS_CSS` into the shadow root's own
  `<style>`. **Render all badge markup inside the `container` it returns:**
  the token variables live on `:root, [data-theme="light"]`, `:root` never
  matches in a shadow tree, so they only exist on/below the container's
  `data-theme="light"`. After changing `packages/tokens`, run
  `pnpm run tokens:sync`; `pnpm run tokens:check` (part of `build` and CI)
  fails on drift between `tokens.json`, `tokens.css`, and the extension copy.
- Accessibility bar (REQUIREMENTS §6), non-negotiable for every badge/popup
  control: keyboard-operable with descriptive accessible labels; no
  assertive ARIA live regions; never steal focus on mount/update; never
  render so it obscures a price, checkout control, or native accessibility
  element. The primary action opens the outbound URL in a new tab — it must
  not navigate the host page away.
- The Marketplace is a Vercel-deployed React app that calls the same
  read-only comparison API (`apps/api`) as the extension — it doesn't fetch
  retailer prices directly, and has no login, checkout, or user tracking.

## 2. Extension MV3 (`apps/extension/**`)

- Content script runs **only** on declared Kroger domains —
  explicit `matches` entries in `manifest.json`, never a broad wildcard.
- One page adapter per retailer, producing a normalized `ProductView`:
  ```ts
  type ProductView = {
    retailer: string;
    canonicalUrl: string;
    productId?: string;
    upc?: string;
    title: string;
    selectedVariant?: string;
    currentPriceCents?: number;
    currency?: string;
    availability: "in-stock" | "out-of-stock" | "unknown";
  };
  ```
- The extension must **never** contain retailer credentials or request
  arbitrary page history. It may send only the current product identity and
  selected retailer/store location to the Pinkless API (`apps/api`) solely to
  obtain a comparison — no other outbound call, no analytics/telemetry.
- Re-evaluate with a **debounced** `MutationObserver` on variant change or
  client-side navigation — debounce so a burst of DOM churn triggers one
  recomputation, not many. Never leave a stale savings badge visible after a
  recompute; if the new state fails any check, remove/hide the badge.
- Render **exactly one** badge, ever, inside a Shadow DOM root, with a fixed
  unique root element ID. On remount, replace the existing root's contents —
  never append a second badge instance.
- Matching/eligibility logic belongs in `packages/matcher` / the API, not
  duplicated inline in the content script.

## 3. API and provider adapters (`apps/api/**`)

- Keep all retailer credentials in Vercel environment variables; never
  expose them to the extension or Marketplace, and never use a client-visible
  `VITE_` prefix for them.
- One provider adapter per retailer (`apps/api/src/providers/**`), each
  returning the shared `Offer` shape — don't leak a provider's raw response
  shape upward.
- Cache by `retailer + product ID/UPC + location + fulfillment method`, with

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hackhersandhims/Pinkless](https://github.com/hackhersandhims/Pinkless) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
