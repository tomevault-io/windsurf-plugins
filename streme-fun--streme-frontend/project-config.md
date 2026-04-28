---
trigger: always_on
description: Streme.fun is a Farcaster-integrated DeFi platform for launching and trading tokens with streaming rewards on Base L2. The platform leverages Superfluid Protocol for real-time token streaming through both Constant Flow Agreements (CFA) for individual streams and General Distribution Agreements (GDA) for group distributions.
---

# Project Overview

Streme.fun is a Farcaster-integrated DeFi platform for launching and trading tokens with streaming rewards on Base L2. The platform leverages Superfluid Protocol for real-time token streaming through both Constant Flow Agreements (CFA) for individual streams and General Distribution Agreements (GDA) for group distributions.

# Tech Stack

- Framework: Next.js 15 (App Router) + TypeScript
- Blockchain: Base L2, wagmi + viem
- Styling: TailwindCSS + DaisyUI
- Auth: Privy (desktop/mobile) + wagmi (Farcaster mini-app)
- APIs: 0x Protocol (gasless swaps), Neynar (Farcaster data)

# Project Structure

```
src/
├── app/          # Next.js pages and API routes
├── components/   # React components
├── hooks/        # Custom hooks (especially useTokenData)
└── lib/          # Utilities, contracts, API helpers
```

# Main Application Pages

- `/` - Token discovery and trading
- `/launch` - Launch new tokens
- `/tokens` - User's token portfolio
- `/token/[address]` - Individual token page
- `/cfa` - Individual streaming (Constant Flow Agreements)
- `/gda` - Group distributions (General Distribution Agreements)
- `/leaderboard` - SUP points leaderboard
- `/crowdfund` - Crowdfunding campaigns

# Key Smart Contracts

- LP Factory: `0xfF65a5f74798EebF87C8FdFc4e56a71B511aB5C8`
- Superfluid Host: `0x4C073B3baB6d8826b8C5b229f3cfdC1eC6E47E74`
- CFA V1: `0x19ba78B9cDB05A877718841c574325fdB53601bb`
- CFA V1 Forwarder: `0xcfA132E353cB4E398080B9700609bb008eceB125`
- GDA V1 Forwarder: `0x6DA13Bde224A05a288748d857b9e7DDEffd1dE08`
- STREME Super Token: `0x3B3Cd21242BA44e9865B066e5EF5d1cC1030CC58`
- STREME Staking Pool: `0xa040a8564c433970d7919c441104b1d25b9eaa1c`
- STREME Staking Address: `0x93419f1c0f73b278c73085c17407794a6580deff`
- STREME Staking Rewards Funder: (Address in useStremeStakingContract)
- Staking Helper: `0x1738e0Fed480b04968A3B7b14086EAF4fDB685A3` (Direct send() for auto-staking)
- Macro Forwarder: `0xFD0268E33111565dE546af2675351A4b1587F89F` (Deprecated)
- Staking Macro V2: `0x5c4b8561363E80EE458D3F0f4F14eC671e1F54Af` (Deprecated)
- Fluid Locker Factory: `0xa6694cab43713287f7735dadc940b555db9d39d9`

# Core Patterns

- Token data managed via `useTokenData` context (centralized balance management)
- API routes follow `/api/[resource]/[action]` pattern
- All addresses lowercase in API calls and subgraph queries
- Token amounts use BigInt for contract interactions
- Timestamps use Firestore format (`_seconds`, `_nanoseconds`) for Firebase data
- Superfluid flow rates are in wei/second, convert using `flowRateToTokensPerDay()`
- Real-time animations use `useStreamingNumber` hook
- Farcaster user data fetched via Neynar API
- Safe image loading with fallback using `SafeImage` component
- StreamingBalance component shows real-time STREME balance with 4 decimals, USD value, and monthly flow rates

# Key Hooks

- `useTokenData` - Centralized token balance management
- `useWallet` - **NEW**: Unified wallet connection hook for both browser and mini-app contexts
- `useUnifiedWallet` - **NEW**: Advanced unified wallet hook with stable address management and auto-connection
- `useAppFrameLogic` - Farcaster mini-app detection and context
- `useStremeBalance` - User's STREME token balance
- `useStremeFlowRate` - User's STREME staking rewards flow rate
- `useCFAFlowRate` - User's net CFA flow rate (incoming - outgoing)
- `useStreamingNumber` - Animated number display for streaming values
- `useDistributionPool` - GDA pool management
- `useCheckin` - Daily check-in functionality (auto-claim after staking)
- `useCheckinModal` - Check-in modal state management
- `useStremeStakingContract` - STREME staking operations (deposit, withdraw, approve)
- `useBestFriendsStreaming` - Farcaster best friends integration

# Mini-App Detection

The app uses a **centralized mini-app detection system** to ensure consistent behavior across all components:

## Detection Utilities (`/src/lib/miniAppDetection.ts`)

- `detectEnvironmentForProviders()` - Early detection for ClientLayout provider selection
- `detectMiniAppWithContext()` - Enhanced detection with Farcaster context for app logic
- `detectMiniApp()` - Core detection function with wallet browser exclusion, then clientFid checks

## Detection Methods (in priority order):
1. **Wallet Browser Exclusion** - Forces desktop mode for all wallet browsers:
   - Coinbase Wallet
   - Rainbow Wallet
   - MetaMask
   - Rabby Wallet
   - Trust Wallet
   - Brave Wallet
   - Opera Crypto Browser
   - OKX Wallet
   - Zerion Wallet
   - 1inch Wallet
2. **Base App clientFid** - Checks for `clientFid === 309857`
3. **Other Farcaster clientFid** - Any other valid Farcaster client

Note: URL-based detection (checking for `/mini` path or `?miniApp=true`) is not used due to reliability concerns - these can be easily manipulated and are not consistently set across different Farcaster clients.

**Important**: All wallet browsers are explicitly excluded from mini-app mode to ensure proper wallet functionality and prevent conflicts.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/streme-fun) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
