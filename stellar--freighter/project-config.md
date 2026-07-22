---
trigger: always_on
description: > Non-custodial Stellar wallet browser extension for Chrome and Firefox.
---

# Freighter

> Non-custodial Stellar wallet browser extension for Chrome and Firefox.
> Monorepo: extension, client SDK (`@stellar/freighter-api`), shared utilities,
> and Docusaurus docs site.

## Glossary

Domain terms you will encounter throughout this codebase:

| Term               | Meaning                                                                            |
| ------------------ | ---------------------------------------------------------------------------------- |
| **Popup**          | Browser extension UI — React app at `extension/src/popup/`, runs in its own page   |
| **Background**     | Service worker at `extension/src/background/` — holds keys, signs, manages storage |
| **Content Script** | Injected into web pages; bridges dApp `window.postMessage` to the extension        |
| **dApp**           | Decentralized app that connects to Freighter via `@stellar/freighter-api`          |
| **XDR**            | Stellar binary serialization format used for transactions and ledger entries       |
| **Soroban**        | Stellar smart contract platform; transactions carry `SorobanData`                  |
| **Blockaid**       | Third-party malicious-transaction scanning service integrated into sign flows      |
| **Horizon**        | Stellar REST API for querying ledger data and submitting transactions              |
| **Mnemonic**       | BIP-39 seed phrase used to derive HD wallet private keys                           |
| **Redux slice**    | Feature-scoped reducer + actions created with `createSlice` (Redux Toolkit)        |
| **i18n key**       | Translation token; all user-facing text must go through `t()` from `react-i18next` |

## Documentation

- [User Guides](https://docs.freighter.app/docs/guide/introduction)
- [API Playgrounds](https://docs.freighter.app/docs/playground/getAddress)
- [Extension Dev Guide](./extension/README.md)
- [E2E Testing Guide](./extension/e2e-tests/README.md)
- [Style Guide](./extension/STYLEGUIDE.MD)
- [Localization](./extension/LOCALIZATION.MD)
- [Hardware Wallet Integration](./extension/INTEGRATING_HARDWARE_WALLET.MD)
- [Soroswap Integration](./extension/INTEGRATING_SOROSWAP.MD)
- [@stellar/freighter-api SDK](./@stellar/freighter-api/README.md)
- [Getting Started](./README.md)
- [Contributing](./CONTRIBUTING.MD)

## Quick Reference

| Item              | Value                                                        |
| ----------------- | ------------------------------------------------------------ |
| Language          | TypeScript, React                                            |
| Node              | >= 22 (`.nvmrc`)                                             |
| Package Manager   | Yarn 4.10.0 (workspaces)                                     |
| State Management  | Redux Toolkit                                                |
| Testing           | Jest (unit), Playwright (e2e)                                |
| Linting           | ESLint flat config + Prettier                                |
| Default Branch    | `master`                                                     |
| Branch Convention | `type/description` (`feature/`, `fix/`, `chore/`, `bugfix/`) |

## Build & Test Commands

```bash
yarn setup                         # Install + allow scripts
yarn start                         # Dev mode (all workspaces)
yarn start:extension               # Extension only
yarn build:extension               # Dev build
yarn build:extension:experimental  # Experimental features enabled
yarn build:extension:production    # Minified production build
yarn build:extension:translations  # Auto-generate translation keys
yarn test                          # Jest watch mode
yarn test:ci                       # Jest CI mode (use before pushing)
yarn test:e2e                      # Playwright e2e
```

## Environment Setup

Create `extension/.env`:

```env
INDEXER_URL=your_backend_v1_prod_url_here
INDEXER_V2_URL=your_backend_v2_prod_url_here
```

## Repository Structure

```
freighter/
├── extension/
│   ├── src/popup/          # React UI
│   ├── src/background/     # Service worker (keys, signing, storage)
│   ├── src/contentScript/  # dApp ↔ extension bridge
│   ├── e2e-tests/          # Playwright tests
│   └── public/             # Static assets, manifest
├── @stellar/freighter-api/ # npm SDK for dApp integration
├── @shared/                # Shared api, constants, helpers
├── docs/                   # Docusaurus site
├── config/                 # Shared Jest/Babel/Webpack configs
└── .github/
    ├── workflows/          # 10 CI/CD workflows
    └── agents/             # Playwright test AI agents (generator, healer, planner)
```

## Architecture

Three runtime contexts communicate exclusively via message passing:

1. **Popup** — React UI, dispatches to background via
   `sendMessageToBackground()` (which wraps `browser.runtime.sendMessage`)
2. **Background** — service worker, processes messages, never directly touches
   the DOM
3. **Content Script** — injected per tab, relays between `window.postMessage`
   and the background via `browser.runtime.sendMessage`

Dev server URLs:

| URL                                 | Purpose                                   |
| ----------------------------------- | ----------------------------------------- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stellar/freighter](https://github.com/stellar/freighter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
