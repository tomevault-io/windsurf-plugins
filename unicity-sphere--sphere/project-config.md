---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Unicity AgentSphere is a React-based cryptocurrency wallet application for the Unicity network. It provides a dual-layer wallet interface supporting both Layer 1 (ALPHA blockchain) and Layer 3 (Unicity state transition network) operations, along with DMs, group chat, and an iframe-based agent system. All wallet operations are handled through `@unicitylabs/sphere-sdk`, with a thin React adapter layer in `src/sdk/`.

## Development Commands

```bash
npm run dev          # Start development server
npm run build        # TypeScript compile + Vite build
npm run lint         # ESLint
npm run test         # Vitest watch mode
npm run test:run     # Vitest single run
npm run preview      # Preview production build
npx tsc --noEmit     # Type check only
```

## Architecture

### Tech Stack
- React 19 + TypeScript ~5.9 with Vite 7
- TanStack Query v5 for server state management
- Tailwind CSS 4 (via `@tailwindcss/vite` plugin)
- Framer Motion for animations
- React Router DOM v7 for routing
- Vitest 4 + jsdom for testing
- `@unicitylabs/sphere-sdk` ^0.5 for all wallet operations (L1, L3, Nostr, IPFS)
- Lucide React for icons
- KaTeX for math rendering

### Routes (App.tsx)

```
/                    — IntroPage (splash screen)
/connect             — ConnectPage (wallet connection)
/home                — HomePage
/agents/:agentId     — AgentPage (dm, group-chat, custom)
/developers          — DevelopersPage (lazy)
/developers/docs     — DocsPage (lazy)
/mine                — MineAlphaPage (lazy)
/markets             — MarketsPage (lazy)
/explore-agents      — AgentsPage (lazy)
/about               — AboutPage (lazy)
```

All routes except `/` and `/connect` are wrapped in `DashboardLayout`.

### Active Agents (src/config/activities.ts)

Only 3 agents are currently enabled:
- `dm` — Messages (private DM via Nostr), requires wallet
- `group-chat` — Group Chat (NIP-29 relay channels), requires wallet
- `custom` — Sphere Agents (load any URL as iframe)

### Provider Tree (main.tsx)

```
StrictMode
  → QueryClientProvider
    → SphereProvider (network="testnet")
      → ServicesProvider (GroupChat)
        → ConnectProvider (wallet connection intents)
          → ThemeInitializer
            → BrowserRouter
              → App
            → ToastContainer
```

### Source Structure

```
src/
├── index.html               # HTML entry point (served via Vite plugin)
├── main.tsx, App.tsx, index.css
├── sdk/                     # React adapter layer over sphere-sdk (24 files)
│   ├── SphereProvider.tsx, SphereContext.ts, types.ts, queryKeys.ts
│   ├── hooks/core/          # useSphere, useWalletStatus, useIdentity, useNametag, useSphereEvents, useIpfsSync
│   ├── hooks/payments/      # useTokens, useBalance, useAssets, useTransfer, useTransactionHistory
│   ├── hooks/l1/            # useL1Balance, useL1Utxos, useL1Send, useL1Transactions
│   ├── hooks/comms/         # useSendDM, usePaymentRequests
│   └── utils/format.ts
├── hooks/                   # 9 app-level hooks
│   ├── useMarketFeed, useTheme, useTutorial, useUIState, useDesktopState
│   └── useGlobalSyncStatus, useVisualViewport, useKeyboardScrollIntoView, useMentionNavigation
├── components/
│   ├── activity/            # IntentIcon, ActivityTicker (market feed via SDK WebSocket)
│   ├── agents/              # AgentCard, IframeAgent, WalletRequiredBlocker
│   ├── chat/                # ChatSection
│   │   ├── dm/              # DMChatSection, DMConversationList, DMMessageList
│   │   ├── group/           # GroupChatSection, GroupList, GroupMessageList
│   │   ├── mini/            # MiniChatWindow, miniChatStore
│   │   ├── hooks/           # useChat, useDmUnreadCount, useGroupChat, useGroupUnreadCount
│   │   ├── data/            # chatTypes (CHAT_KEYS, GROUP_CHAT_KEYS)
│   │   └── utils/           # avatarColors, groupChatHelpers
│   ├── wallet/
│   │   ├── L1/              # L1WalletModal, VestingDisplay, BridgeModal, modals
│   │   ├── L3/              # L3WalletView, modals, currency utils
│   │   ├── onboarding/      # CreateWalletFlow, hooks
│   │   ├── shared/          # Shared wallet components, modals, hooks
│   │   └── ui/              # BaseModal, Button, ModalHeader, AlertMessage, EmptyState, MenuButton
│   ├── layout/              # DashboardLayout, Header, IpfsSyncIndicator
│   ├── desktop/             # DesktopLayout, TabBar, Taskbar, DesktopShortcuts
│   ├── connect/             # ConnectIntentHandler, ConnectionApprovalModal, ConnectProvider
│   ├── splash/              # SplashScreen
│   ├── theme/               # ThemeInitializer, ThemeToggle
│   ├── tutorial/            # TutorialOverlay
│   └── ui/                  # ComingSoonModal, Toast, toast-utils
├── pages/                   # 10 page components (see Routes)
├── contexts/                # ServicesContext, ServicesProvider, useServices
├── services/                # marketplaceApi, userApi
├── config/
│   ├── activities.ts        # Agent definitions (AgentConfig, getAgentConfig)
│   └── storageKeys.ts       # All localStorage key constants
├── lib/queryClient.ts       # TanStack Query client

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unicity-sphere/sphere](https://github.com/unicity-sphere/sphere) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-28 -->
