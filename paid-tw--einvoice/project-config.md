---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A pnpm monorepo publishing a unified **Taiwan e-invoice SDK** (`@paid-tw/einvoice`) plus per-provider adapter packages. Every Taiwan value-added center wraps the same 財政部 MIG 4.0 spec, so the core models the five operations once — **issue (開立) / void (作廢) / allowance (折讓) / void-allowance (折讓作廢) / query (查詢)** — and each provider is a thin adapter mapping the unified model ⇄ its wire format.

## Commands

```bash
pnpm install
pnpm build          # build all packages via tsdown (rolldown — ESM + CJS + d.ts)
pnpm test           # vitest run (offline; uses MSW mocks)
pnpm test:watch     # vitest watch
pnpm typecheck      # tsc --noEmit across packages
pnpm lint           # oxlint --type-aware (oxc linter)
pnpm format         # oxfmt --write   (oxc formatter, printWidth 100)
pnpm format:check   # oxfmt --check   (CI gate; fails if anything is unformatted)
```

**Before pushing, run what CI runs:** `pnpm build && pnpm typecheck && pnpm lint && pnpm format:check && pnpm test`. CI (`.github/workflows/ci.yml`) has two jobs: a **build job** (Node 22 — `build` → `typecheck` → `lint` → `format:check` → `check:exports`) and a **test matrix** (Node 18/20/22/24, offline tests only; tests resolve workspace packages to source via the vitest alias, so they need no build). `pnpm build` includes the **`.d.ts` declaration build** (tsdown — a distinct type check that can fail even when `pnpm test` is green, e.g. a public-API/arity issue that only surfaces in the declaration build). The build toolchain (tsdown/rolldown) needs **Node ≥22**, distinct from the library's runtime floor of `engines >=18` (consumers run the prebuilt dist). Don't infer a green CI from `pnpm test` alone — and don't mask the build's exit status (a silenced `pnpm build` that falls through to a later `|| echo OK` hides a real failure).

Run a single package's tests / a single file / a single test:
```bash
pnpm --filter @paid-tw/einvoice-amego exec vitest run
pnpm exec vitest run packages/einvoice-amego/src/__tests__/unified.test.ts
pnpm exec vitest run -t "issue"        # by test name
```

**Live tests** hit real provider sandboxes and are skipped unless gated env vars are set (`AMEGO_LIVE=1`, `ECPAY_LIVE=1`, `EZPAY_LIVE=1`, etc.). CI runs offline only. Example:
```bash
AMEGO_LIVE=1 pnpm --filter @paid-tw/einvoice-amego exec vitest run live
```

## Architecture

```
@paid-tw/einvoice (core)     provider-agnostic: types, InvoiceProvider, Zod schemas, MockProvider
        ▲ implements InvoiceProvider
        │
@paid-tw/einvoice-amego      maps unified model ⇄ Amego wire format (MD5 sign)
@paid-tw/einvoice-ecpay      ECPay B2C 2.0 (AES)
@paid-tw/einvoice-ezpay      ezPay 藍新 (AES)
@paid-tw/einvoice-ezpay-crossborder   ezPay 境外電商 (cross-border B2C)
@paid-tw/einvoice-ezreceipt  ezReceipt 易發票 (order-oriented REST, token auth)
```

Adapters depend on core via `workspace:*` and list it as a tsdown `external` — they never bundle it. Install only the adapter you use; adapters don't pull in each other's deps.

**Core helpers adapters reuse** (all exported from `@paid-tw/einvoice`): `parseInput` (schema → validated input, raising `InvoiceError`), `taxTypeToCode` (unified `TaxType` → MIG `1`/`2`/`3`), `parseTaipeiDate`/`taipeiDateTime` (Asia/Taipei date ⇄ wire string), `tracedFetch` (the debug-logging fetch wrapper), and the amount helpers `composeTaxExclusive`/`splitTaxInclusive`. Prefer these over re-implementing per adapter.

**Core is the contract.** Application code depends only on `InvoiceProvider` (`packages/einvoice/src/provider.ts`) and the unified types — never on a concrete adapter. Switching providers means swapping the constructor (`createAmegoProvider(...)` → `createEcpayProvider(...)`), nothing else.

### Key invariants when working on adapters

- **Money is integer TWD — except cross-border foreign currency.** For TWD (the default, and every domestic provider) the statutory amount fields (`salesAmount`/`taxAmount`/`totalAmount`) are integers in New Taiwan Dollars — a MIG invariant. **Exception:** the cross-border adapter (`@paid-tw/einvoice-ezpay-crossborder`) accepts *2-decimal foreign amounts* in the unified input when `currency` ≠ TWD (see `fmtAmount` in its `provider.ts`: `foreign ? value.toFixed(2) : Math.round(value)`); the government filing is still in TWD, derived server-side from `exchangeRate`. So `currency` (ISO 4217) + `exchangeRate` *annotate* the sale, and on cross-border they also imply the wire amounts are decimal foreign-currency values, not integer TWD.
- **Capabilities are declared, not discovered.** Each provider exposes a `capabilities: ReadonlySet<Capability>` (`packages/einvoice/src/capabilities.ts`). Callers feature-detect with `supports()` / `assertSupports()`. A provider lacking `FOREIGN_CURRENCY` must **reject** a non-TWD `currency` (throw `UNSUPPORTED`), not silently drop it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paid-tw/einvoice](https://github.com/paid-tw/einvoice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
