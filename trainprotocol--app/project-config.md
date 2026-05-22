---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Dev Commands

```bash
pnpm dev                              # Start Next.js dev server (apps/app)
pnpm build                            # Build all packages first, then the app
pnpm build:sdk                        # Build @train-protocol/sdk only
pnpm build:packages                   # Build all workspace packages
pnpm --filter train-app build:workers # Compile worker bundles (tsconfig.worker.json)

# SDK development
pnpm --filter @train-protocol/sdk dev          # Watch mode for SDK
pnpm --filter @train-protocol/sdk check:types  # Type-check SDK
```

The app has no `lint` script and no in-app test setup — tests live in packages (vitest), notably `packages/sdk` (`pnpm --filter @train-protocol/sdk test`). Node.js >=20.9.0 required. Package manager: pnpm 10.20.0.

## Architecture

**Monorepo** (pnpm workspaces — `apps/*` and `packages/**`):
- `apps/app` — Next.js 15.5 frontend (App Router). Entry: `app/layout.tsx` (server, awaits `getSettings()`) → `app/providers.tsx` (consolidated client providers). The provider tree is split: outer (PostHog, SWR, Theme, Intercom) and inner `AppShell` (Settings, Tooltip, Train, Wallets, ThemeWrapper, ErrorBoundary, SwapAccounts, AsyncModal). The persistent shell (`ThemeWrapper` — sidebar/navbar/footer) is rendered *above* the Suspense boundary so it never blanks. Only `<QueryProvider>` (which reads `useSearchParams` internally) plus `AuthDialog`, `SwapModalRoot`, and the page `children` sit inside `<Suspense fallback={null}>`. Route segments: `app/{page,swap,settings,transactions,nocookies}/page.tsx`. Root `error.tsx` + `not-found.tsx`. `middleware.ts` sets `Cache-Control: public, s-maxage=60, stale-while-revalidate`. The layout is no longer `force-dynamic` — `getSettings` is cached via `unstable_cache` (60s revalidate) and all routes build as `○ Static`. Sidebar `<Link>`s use Next 15 default prefetch (eager prefetch on viewport entry for static routes); the logo opts out via `prefetch={false}` since it points at `/`. All four sidebar nav targets (logo, Home, History, Settings) preserve persistent embed query params via `buildHrefWithPersistantParams`. Web Workers live in `apps/app/workers/` (e.g. `helios` light client) and are built via `build:workers`.
- `apps/packagesdemo` — small Next.js Pages-Router demo app for exercising published SDK packages.
- `packages/sdk` — `@train-protocol/sdk`: core HTLC protocol logic, `TrainApiClient`, lock verification, consensus helpers (vitest tests in `__tests__/`).
- `packages/blockchains/` — chain-specific HTLC client implementations: `evm`, `solana`, `starknet`, `tron`, `aztec`, plus shared `utils`.
- `packages/auth` — `@train-protocol/auth` shared auth helpers.
- `packages/react` — `@train-protocol/react` shared React utilities.

**What the app does**: Cross-chain atomic swaps using HTLC (Hash Time-Locked Contracts). Users lock funds on a source chain, a solver locks on the destination chain, then secrets are revealed to complete the swap. EVM is the primary chain; Solana, Starknet, TON, Aztec, Tron, and Fuel support is in progress (see `FUEL_PHTLC.json`).

### State Management
- **Zustand stores** (`apps/app/stores/`): `swapStore` (main swap state), `walletStore`, `addressesStore`, `balanceStore`, `rpcConfigStore`, `authDialogStore`, `aztecWalletStore`, `starknetWalletStore`, `contractWalletsStore`, `recentRoutesStore`, `routeSortingStore`, `routeTokenSwitchStore`, `usdModeStore`. There is no `secretDerivationStore` — secret derivation lives in helpers/components.
- **React Context** (`apps/app/context/`): `swapAccounts` (wallet/account handling), `formWizardProvider` (multi-step forms), `walletHookProviders` (per-chain wallet hook composition), `query` (React Query), `settings`, `snapPointsContext`, `timerContext`, `asyncModal`. HTLC contract interactions and EVM connectors are no longer separate contexts — they're handled inline via wallet hooks and the SDK's HTLC clients.

### Layout & navigation
- `ThemeWrapper` (inside providers) renders the persistent shell: left-side `AppSidebar` + top app-header strip (desktop) that holds `<PendingSwap />` + content area + `GlobalFooter`.
- `HeaderWithMenu` lives inside `Widget` (not the app shell) and contains back button, mobile-only wallet/menu cluster. On mobile `PendingSwap` also appears here (no top app-header on mobile).
- **Route loading skeletons**: `app/{settings,transactions,swap,faucet}/loading.tsx` each inline a centered `Loader2` spinner (identical body — `flex items-center justify-center w-full min-h-93.5` wrapper around `<Loader2 className="h-10 w-10 text-primary animate-spin" />`). `components/Loading.tsx` holds the same body for non-route Suspense boundaries. Copy the body into a new `loading.tsx` when adding a route that needs a Suspense fallback; let it diverge if that route eventually wants a different skeleton.
- **Maintenance fallback**: when `getSettings()` returns `null`, `app/providers.tsx` renders `<MaintananceContent />` in place of the full provider tree (inside `IntercomProvider` so `useIntercom` works). Root layout does NOT call `notFound()`.

### API Layer — Station API

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TrainProtocol/app](https://github.com/TrainProtocol/app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
