---
trigger: always_on
description: This document provides comprehensive guidelines for working with the Skip Go Explorer codebase.
---

# Skip Go Explorer - Agent Guide

This document provides comprehensive guidelines for working with the Skip Go Explorer codebase.

---

## 1. Project Overview

### What does this project do?

Skip Go Explorer is a **cross-chain transaction explorer** built with Next.js. It allows users to:
- Track cross-chain transactions across multiple blockchain ecosystems
- View detailed transaction status and transfer events
- Visualize transaction routing through different chains
- Monitor asset releases and transfers in real-time
- Debug transactions with raw data inspection

### Tech Stack

| Category | Technology |
|----------|-----------|
| Framework | Next.js 15.4 (App Router) |
| Runtime | React 19.1 |
| Language | TypeScript 5 |
| State Management | Jotai 2.13 + Jotai-TanStack-Query 0.11 |
| Data Fetching | TanStack Query (React Query) 5.84 |
| Styling | styled-components (from widget package) |
| URL State | nuqs 2.4 |
| API Client | @skip-go/client (workspace monorepo package) |
| Error Boundaries | react-error-boundary 6.0 |
| Utilities | @uidotdev/usehooks 2.4 |

### Main Applications/Services

This is a **single-page application (SPA)** that serves as a transaction explorer interface for Skip Protocol's cross-chain infrastructure.

---

## 2. Project Structure

```
apps/explorer/
├── src/
│   ├── app/                          # Next.js App Router pages
│   │   ├── page.tsx                  # Main transaction explorer page
│   │   ├── layout.tsx                # Root layout with metadata
│   │   ├── template.tsx              # Client-side providers wrapper
│   │   └── globals.css               # Global styles
│   ├── components/                   # UI components
│   │   ├── Badge.tsx                 # Status badges
│   │   ├── Bridge.tsx                # Bridge visualization
│   │   ├── ChainSelector.tsx         # Chain selection dropdown
│   │   ├── ErrorCard.tsx             # Error display
│   │   ├── Navbar.tsx                # Navigation bar
│   │   ├── QueryProvider.tsx         # React Query provider
│   │   ├── SearchButton.tsx          # Search trigger
│   │   ├── SuccessfulTransactionCard.tsx  # Success state
│   │   ├── TokenDetails.tsx          # Token information display
│   │   ├── TransactionDetails.tsx    # Transaction metadata
│   │   ├── TransferEventCard.tsx     # Individual transfer event
│   │   ├── TxHashInput.tsx           # Transaction hash input
│   │   └── modals/
│   │       ├── SearchModal.tsx       # Mobile search modal
│   │       └── ViewRawDataModal.tsx  # Raw JSON viewer
│   ├── constants/
│   │   ├── chainIdsSortedToTop.ts    # Chain priority sorting
│   │   └── modal.tsx                 # Modal identifiers
│   ├── hooks/                        # Custom React hooks
│   │   ├── useGetTransferAssetReleaseAsset.ts
│   │   ├── useOverallStatusLabelAndColor.ts
│   │   ├── useTheme.ts               # Theme detection
│   │   └── useTransactionHistoryItemFromUrlParams.ts
│   ├── icons/                        # SVG icon components
│   │   ├── BridgeIcon.tsx
│   │   ├── ClockIcon.tsx
│   │   ├── CoinsIcon.tsx
│   │   ├── RightArrowIcon.tsx
│   │   ├── SearchIcon.tsx
│   │   └── TopRightArrowIcon.tsx
│   ├── jotai.ts                      # Jotai re-exports
│   ├── mixins/
│   │   └── styledScrollbar.ts        # Scrollbar styling
│   ├── types/
│   │   └── theme.d.ts                # Theme type definitions
│   └── utils/
│       ├── denomUtils.ts             # Denomination transformations
│       └── skipClientConfig.ts       # Skip API client setup
├── public/                           # Static assets
│   ├── logo.svg                      # Dark theme logo
│   ├── logo-light.svg                # Light theme logo
│   ├── dark-bg.svg                   # Dark background
│   ├── light-bg.svg                  # Light background
│   └── skip-*.png                    # Favicons & manifest icons
├── declarations.d.ts                 # TypeScript declarations
├── eslint.config.mjs                 # ESLint configuration
├── next.config.ts                    # Next.js configuration
├── package.json                      # Dependencies & scripts
└── tsconfig.json                     # TypeScript configuration
```

### Key Configuration Files

| File | Purpose |
|------|---------|
| `next.config.ts` | Next.js configuration, webpack aliases, remote image patterns |
| `tsconfig.json` | TypeScript compiler options, path aliases to widget package |
| `eslint.config.mjs` | Linting rules for Next.js and TypeScript |
| `package.json` | Dependencies, scripts, workspace references |

### Entry Points

1. **Application Entry**: `src/app/layout.tsx` - Root layout with metadata
2. **Main Page**: `src/app/page.tsx` - Transaction explorer UI
3. **Client Providers**: `src/app/template.tsx` - Jotai, React Query, NiceModal providers
4. **API Client**: `src/utils/skipClientConfig.ts` - Skip client initialization

---

## 3. Build Commands

All commands should be run from the **monorepo root** or from the `apps/explorer` directory:

| Command | Description | Location |
|---------|-------------|----------|
| `yarn dev` | Start development server (http://localhost:3000) | `apps/explorer` |
| `yarn build` | Build production bundle | `apps/explorer` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skip-mev/skip-go](https://github.com/skip-mev/skip-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
