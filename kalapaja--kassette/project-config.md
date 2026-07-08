---
trigger: always_on
description: Kassette is the payment page SPA for Kalatori. Merchants embed it (or serve it from their Kalatori daemon at `/public/`). It renders an invoice, connects a wallet via Reown AppKit/wagmi, and orchestrates token swaps (Uniswap) or cross-chain bridges (Across) to pay in the invoice's target asset. No backend of its own — all state comes from the Kalatori daemon's HTTP API and on-chain reads.
---

# Kassette — AI Agent Guide

Kassette is the payment page SPA for Kalatori. Merchants embed it (or serve it from their Kalatori daemon at `/public/`). It renders an invoice, connects a wallet via Reown AppKit/wagmi, and orchestrates token swaps (Uniswap) or cross-chain bridges (Across) to pay in the invoice's target asset. No backend of its own — all state comes from the Kalatori daemon's HTTP API and on-chain reads.

## How We Work

### Development Policy

- **Dagger for all CI**: use `dagger call checks` for fast validation, `dagger call end-to-end` for E2E, or run individual checks. Pipeline defined in TypeScript at `.dagger/src/index.ts`
- **Layer-cached builds**: Dagger caches pnpm install separately from source. Only code changes trigger rebuilds, not dependency re-downloads
- **Git hooks**: lefthook auto-installs via `pnpm install` (the `prepare` script). Pre-commit runs lint + format on staged files; pre-push runs typecheck + tests + tag version check
- **Conventional commits**: enforced by commitlint in the commit-msg hook

### Documentation Policy

- **Done something — write it down.** Every architectural decision, troubleshooting finding, or pattern change gets recorded in `docs/`.
- When a doc contradicts code, ask the user which is correct and update the other.
- See [docs/doc-update-triggers.md](docs/doc-update-triggers.md) for the mandatory update checklist.

### Research Policy

- **Never assume library APIs from memory** — look up in Context7 or Exa first.

### Trust Hierarchy

When sources disagree, trust in this order:

1. **Code, build files, CI workflows** — canonical source of truth
2. **Tests** — verify behavior claims
3. **Specific docs** (e.g., `docs/testing-strategy.md`) override general docs (e.g., `AGENTS.md`)
4. **Docs describe intent and patterns** — not guaranteed implementation truth
5. When unsure, **ask the user** which is correct and update the other

### Editing Strategy

- Prefer minimal, surgical edits. Don't refactor adjacent code opportunistically.
- Add or update tests when behavior changes.
- Check [docs/doc-update-triggers.md](docs/doc-update-triggers.md) after changes.

### Writing Style

- Context-aware, terse, informative, concise.
- No unnecessary abstractions — three similar lines > premature helper.

## Commands

```bash
# Dagger (preferred — reproducible, cached):
dagger call checks                  # lint + format + typecheck + test + audit + build (~60s)
dagger call lint                    # ESLint only
dagger call format-check            # Prettier only
dagger call typecheck               # tsc --noEmit
dagger call test                    # Vitest with coverage thresholds
dagger call audit                   # pnpm audit (blocking on critical)
dagger call audit-advisory          # pnpm audit (high/moderate, advisory)
dagger call build                   # Production Angular build → dist/browser
dagger call end-to-end              # Playwright E2E against production build
dagger call release-zip --version=X.Y.Z  # SRI-patched release ZIP

# Local pnpm (same commands Dagger runs):
pnpm dev                            # Dev server on :3001 with MSW mocks
pnpm build                          # Production build
pnpm test                           # Vitest
pnpm test:coverage                  # Vitest with coverage + thresholds
pnpm lint                           # ESLint (zero warnings)
pnpm lint:fix                       # ESLint autofix
pnpm format                         # Prettier write
pnpm format:check                   # Prettier check
pnpm e2e                            # Playwright (starts dev server automatically)
pnpm e2e:ui                         # Playwright interactive UI mode

# Release:
pnpm release:tag                    # Create signed tag from package.json version
```

## Architecture

- **Entry point:** `src/main.ts` — bootstraps Angular app, starts MSW iff `environment.mocks`. `environment.ts` (dev) sets `mocks: true`; `environment.prod.ts`, `environment.e2e.ts`, and `environment.no-mocks.ts` set `mocks: false`. The `no-mocks` build config pairs with `pnpm dev:no-mocks` for manual testing against a real backend without production optimizations. E2E uses `production: true` so the bundle ships without MSW — Playwright handles mocking at the network layer instead.
- **Root component:** `src/app/app.component.ts` — shell with `<router-outlet>`
- **Main page:** `src/app/pages/payment/payment-layout.component.ts` — THE main component with all step rendering via `@switch`
- **Components:** `src/app/components/` — 11 standalone Angular components, prefixed with `kp-` (e.g., `kp-button`, `kp-input`, `kp-webview-escape`)
- **Services:** `src/app/services/` — 14 injectable services (AppKit, WalletState, PaymentState, Invoice, Balance, Payment, Price, Token, Quote, Uniswap, Across, PendingTx, Translation, Layout)
- **Config:** `src/app/config/` — chains, tokens, uniswap, across (plain TypeScript modules)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kalapaja/Kassette](https://github.com/Kalapaja/Kassette) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
