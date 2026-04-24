---
trigger: always_on
description: Plaid competitor — an Expo SDK that runs an embedded browser to log into banking sites, extract account data (accounts, routing/account numbers, transactions), and shows a live minimized visual preview of the browser with status captions explaining each step.
---

# Conduit SDK — Developer Guide

Plaid competitor — an Expo SDK that runs an embedded browser to log into banking sites, extract account data (accounts, routing/account numbers, transactions), and shows a live minimized visual preview of the browser with status captions explaining each step.

## Commands

```bash
npm install           # Install dependencies
npm run build         # Compile TypeScript to dist/
npm test              # Run all tests
npm run test:watch    # Run tests in watch mode
npm run test:coverage # Run tests with coverage report
npm run typecheck     # Type-check without emitting
npm run lint          # Lint with ESLint
npm run lint:fix      # Lint and auto-fix
npm run format        # Format with Prettier
npm run format:check  # Check formatting without writing
npm run clean         # Remove dist/
```

## Project Architecture

```
src/
├── adapters/                # Bank-specific adapter implementations (per-bank automation)
│   ├── banks/               # Built-in bank adapter configurations
│   │   ├── bank-of-america.ts
│   │   ├── chase.ts
│   │   ├── index.ts         # Barrel export for built-in adapters
│   │   └── wells-fargo.ts
│   ├── index.ts             # Public API re-exports
│   ├── registry.ts          # BankAdapterRegistry — plugin registration with duplicate/conflict detection
│   ├── types.ts             # Adapter types: selectors, extractors, MFA detection, config
│   └── validation.ts        # Config validation with detailed error messages
├── auth/                    # Bank authentication module
│   ├── types.ts             # All auth types (discriminated unions, error types, validation)
│   ├── auth-state-machine.ts # State machine enforcing valid auth transitions
│   ├── auth-module.ts       # Main orchestrator for auth flow
│   ├── mfa-handler.ts       # MFA challenge/response loop handler
│   └── index.ts             # Public API re-exports
├── browser/                 # Browser automation interface (port/adapter boundary)
│   ├── types.ts             # BrowserDriver interface + result types
│   └── index.ts             # Public API re-exports
├── core/                    # Embedded browser engine (WebView integration)
│   ├── BrowserEngine.ts     # Main engine: navigation, JS injection, DOM extraction
│   ├── MessageBridge.ts     # RN ↔ WebView communication bridge with bridge injection script
│   ├── CookieManager.ts     # Cookie storage, domain filtering, persistence
│   └── index.ts             # Public API re-exports
├── types/                   # Shared type definitions
│   ├── conduit.ts           # Core domain types: Account, Transaction, BankAdapter, ConduitConfig, LinkSession
│   ├── navigation.ts        # Navigation state machine (discriminated union, transitions)
│   ├── bridge.ts            # WebView message types (inbound/outbound), WebViewRef, CookieData
│   └── index.ts             # Barrel export
├── extractors/              # Data extraction from bank DOM pages (CDT-13)
│   ├── account-extractor.ts # Account extraction: transforms, type inference, assembly
│   └── index.ts             # Barrel export
├── sdk/                     # High-level SDK types for host app integration
│   ├── types.ts             # PreviewState, PreviewStatus — host-facing preview state
│   └── index.ts             # Barrel export
├── ui/                      # Preview UI components (browser preview, status captions)
│   ├── BankSelector.ts      # Searchable bank list controller (headless, framework-agnostic)
│   ├── ConduitPreview.ts    # React component factory for bank browser preview
│   ├── types.ts             # UI component types (ConduitPreviewProps, PreviewRenderInfo)
│   ├── preview/             # Visual browser preview module (CDT-4)
│   │   ├── types.ts         # Preview types: dimensions, positions, transitions, masking config
│   │   ├── browser-preview-controller.ts  # Headless controller orchestrating preview state
│   │   ├── browser-preview-component.ts   # React component factory for live browser preview
│   │   ├── style-utilities.ts             # CSS style computation (scaling, positioning, transitions)
│   │   ├── sensitive-field-masker.ts      # JS injection for blurring sensitive fields
│   │   ├── transition-state-machine.ts    # Page transition animation state machine
│   │   └── index.ts         # Barrel exports
│   └── index.ts             # UI module barrel exports
└── index.ts                 # SDK entry point

server/
├── server.ts                  # Live testing server (Puppeteer + Express)
├── stealth.ts                 # Anti-detection stealth module (CDT-10)
└── package.json               # Server-specific dependencies

tests/
├── auth/
│   ├── types.test.ts              # Validation and error type tests
│   ├── auth-state-machine.test.ts # State machine transition tests
│   ├── auth-module.test.ts        # Integration tests with mock browser
│   └── mfa-handler.test.ts        # MFA flow tests
├── adapters/
│   ├── banks.test.ts              # Built-in adapter config tests (Chase, BofA, Wells Fargo)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eastseymour) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
