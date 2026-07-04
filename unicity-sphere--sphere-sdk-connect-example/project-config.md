---
trigger: always_on
description: Demonstration project showing how to integrate the **Sphere Connect** protocol into browser and Node.js applications. The Connect module enables dApps to interact with Sphere wallets through a transport-agnostic, permission-based RPC interface.
---

# CLAUDE.md - Sphere SDK Connect Example

Demonstration project showing how to integrate the **Sphere Connect** protocol into browser and Node.js applications. The Connect module enables dApps to interact with Sphere wallets through a transport-agnostic, permission-based RPC interface.

## Project Structure

```
sphere-sdk-connect-example/
├── browser/                    # React dApp example (Vite + Tailwind)
│   ├── src/
│   │   ├── App.tsx            # Sidebar navigation + section routing
│   │   ├── main.tsx           # React entry point
│   │   ├── hooks/
│   │   │   └── useWalletConnect.ts   # Core hook: popup/iframe connect logic
│   │   ├── lib/
│   │   │   ├── types.ts             # Local TS interfaces (Asset, Token, etc.)
│   │   │   └── format.ts           # Amount formatting (decimals, fiat, truncate, relativeTime)
│   │   └── components/
│   │       ├── ConnectButton.tsx     # "Connect Wallet" button with loading state
│   │       ├── layout/
│   │       │   ├── PageShell.tsx     # Sidebar + header + content area layout
│   │       │   └── WalletHeader.tsx  # Compact header: nametag, address, disconnect
│   │       ├── ui/
│   │       │   ├── ResultDisplay.tsx # JSON result viewer with copy + raw toggle
│   │       │   ├── CoinBadge.tsx    # Token icon (img or colored letter) + symbol
│   │       │   ├── CoinSelect.tsx   # Dropdown token selector (fetches assets from wallet)
│   │       │   └── StatusBadge.tsx  # Colored status badges (confirmed/pending/failed)
│   │       ├── queries/             # 6 query panels (read-only operations)
│   │       │   ├── IdentityPanel.tsx     # sphere_getIdentity
│   │       │   ├── AssetsPanel.tsx       # sphere_getAssets (table with icons, fiat, 24h)
│   │       │   ├── BalancePanel.tsx      # sphere_getBalance + sphere_getFiatBalance
│   │       │   ├── TokensPanel.tsx       # sphere_getTokens (with status badges)
│   │       │   ├── HistoryPanel.tsx      # sphere_getHistory (with type badges)
│   │       │   └── ResolvePanel.tsx      # sphere_resolve (identifier input)
│   │       ├── intents/             # 6 intent panels (require wallet approval)
│   │       │   ├── SendPanel.tsx         # send (recipient, amount, coin selector, memo)
│   │       │   ├── MintPanel.tsx         # mint (coinId, amount)
│   │       │   ├── DMPanel.tsx           # dm (recipient, message)
│   │       │   ├── PaymentRequestPanel.tsx # payment_request (recipient, amount, coin, message)
│   │       │   ├── ReceivePanel.tsx      # receive (button only, no params)
│   │       │   └── SignMessagePanel.tsx  # sign_message (message textarea)
│   │       └── events/
│   │           └── EventLogPanel.tsx # Color-coded, filterable event log
│   ├── index.html
│   ├── package.json
│   ├── tsconfig.json
│   └── vite.config.ts
│
└── nodejs/                    # Node.js CLI example
    ├── src/
    │   ├── index.ts               # Interactive CLI client (all queries + intents)
    │   └── mock-wallet-server.ts  # Mock wallet with rich test data
    ├── package.json
    └── tsconfig.json
```

## Quick Start

### Browser dApp

```bash
cd browser
npm install
npm run dev        # Vite dev server on http://localhost:5174
```

Requires a wallet app running at `http://localhost:5173` (the Sphere wallet).

### Node.js CLI

```bash
cd nodejs
npm install

# Terminal 1: Start mock wallet server
npm run server     # WebSocket server on ws://localhost:8765

# Terminal 2: Run CLI client
npm run client     # Connects to ws://localhost:8765
```

CLI commands:
- **Queries:** `identity`, `balance`, `assets`, `fiat`, `tokens`, `history`, `resolve @tag`
- **Intents:** `send @to amt [coin]`, `mint <coinId> <amount>`, `dm @to message`, `pay @to amt [coin] [message]`, `receive`, `sign message text`
- **Other:** `disconnect`, `help`

## Dependencies

Both subprojects pin a published sphere-sdk version:
```json
"@unicitylabs/sphere-sdk": "0.10.2"
```

- **Browser:** React 19, Vite 7, Tailwind CSS 4
- **Node.js:** `ws` (WebSocket), `tsx` (TypeScript runner)

## Sphere Connect Protocol Overview

### Architecture

```
┌──────────┐                          ┌──────────────┐
│   dApp   │   ConnectClient          │    Wallet     │   ConnectHost
│          │◄─────────────────────────►│   (Sphere)   │
│          │   Transport layer:       │              │
│          │   - PostMessage (browser) │              │
│          │   - WebSocket (Node.js)  │              │
└──────────┘                          └──────────────┘
```

### Key Imports

```typescript
// Core protocol
import { ConnectClient, ConnectHost, RPC_METHODS, INTENT_ACTIONS, PERMISSION_SCOPES }
  from '@unicitylabs/sphere-sdk/connect';

// Browser transport (iframe/popup communication)
import { PostMessageTransport }
  from '@unicitylabs/sphere-sdk/connect/browser';

// Node.js transport (WebSocket)
import { WebSocketTransport }
  from '@unicitylabs/sphere-sdk/connect/nodejs';

// Types
import type { PublicIdentity, DAppMetadata, PermissionScope, ConnectResult }
  from '@unicitylabs/sphere-sdk/connect';
```

### TypeScript Path Aliases (Browser)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unicity-sphere/sphere-sdk-connect-example](https://github.com/unicity-sphere/sphere-sdk-connect-example) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
