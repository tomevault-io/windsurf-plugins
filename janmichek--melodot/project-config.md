---
trigger: always_on
description: _Comprehensive instructions for AI agents helping develop Melodot_
---

# Melodot Development Guide

_Comprehensive instructions for AI agents helping develop Melodot_

## Project Overview

**Melodot** is a decentralized music discovery and tipping platform that:
- Records audio using Web Audio API
- Identifies songs through audio recognition
- Enables direct artist tipping via smart contracts on Polkadot
- Allows artists to claim their identity and withdraw donations
- Integrates Web3Auth for seamless user onboarding

## Tech Stack

### Core Technologies
- **Smart Contracts**: Solidity 0.8.28, Hardhat, Polkadot Paseo Asset Hub
- **Frontend**: React 18, TypeScript, Vite
- **Web3**: Wagmi, Viem, Web3Auth Modal
- **Styling**: TailwindCSS 4, Radix UI, Lucide icons
- **Audio**: Web Audio API
- **API**: Vercel serverless functions
- **Package Manager**: Bun
- **Testing**: Playwright (e2e)

### Key Dependencies
```json
{
  "wagmi": "Web3 React hooks",
  "viem": "TypeScript Web3 client",
  "@web3auth/modal": "Social login for Web3",
  "@tanstack/react-query": "Data fetching",
  "lucide-react": "Icon library",
  "@radix-ui": "Headless UI components"
}
```

## Project Structure

```
shaz/
├── contracts/              # Smart contracts
│   ├── contracts/
│   │   └── Donate.sol     # Artist donation contract
│   ├── hardhat.config.ts  # Hardhat configuration
│   ├── test/              # Contract tests
│   └── ignition/          # Deployment configs
├── api/                   # Vercel serverless functions
│   ├── endpoints/
│   │   ├── discover.ts   # Music discovery endpoint
│   │   ├── artist.ts      # Single artist info
│   │   ├── artists.ts     # Multiple artists info
│   │   ├── track.ts       # Track information
│   │   └── verify.ts      # Artist verification
│   ├── api.ts             # Centralized API utilities
│   ├── mock-discovery-data.ts
│   └── utils.ts           # API utilities
├── src/                   # React application
│   ├── components/        # React components
│   │   ├── DiscoveryButton.tsx
│   │   ├── DiscoveryRecorder.tsx
│   │   ├── DiscoveryCard.tsx
│   │   ├── DonationForm.tsx
│   │   ├── OwnerWithdrawForm.tsx
│   │   ├── ArtistPayoutForm.tsx
│   │   ├── ClaimCard.tsx
│   │   ├── ClaimFlow.tsx
│   │   ├── UserMenu.tsx
│   │   ├── AppSidebar.tsx
│   │   ├── Header.tsx
│   │   ├── StatisticsDashboard.tsx
│   │   └── ui/            # Radix UI components
│   ├── pages/            # Page components
│   │   ├── Discover.tsx   # Main discovery page
│   │   ├── Claim.tsx      # Artist claim page
│   │   ├── Donations.tsx  # Donations history
│   │   └── Owner.tsx      # Owner dashboard
│   ├── hooks/            # Custom React hooks
│   ├── lib/              # Utilities
│   ├── generated.ts      # Auto-generated Wagmi types
│   └── wagmi-config.ts   # Wagmi configuration
├── tests/                # Playwright e2e tests
├── dist/                 # Production build
└── package.json
```

## Network Configuration

### Paseo Asset Hub Testnet

**Network Details:**
- Chain ID: `420420417` (0x190f1b41)
- RPC: `https://services.polkadothub-rpc.com/testnet`
- Explorer: `https://blockscout-testnet.polkadot.io`
- Faucet: `https://faucet.polkadot.io/`
- Currency: PAS

**Hardhat Configuration:**
```typescript
import { HardhatUserConfig } from "hardhat/config";
import "@nomicfoundation/hardhat-toolbox";
import "@parity/hardhat-polkadot";

const config: HardhatUserConfig = {
  solidity: "0.8.28",
  resolc: {
    compilerSource: "npm",
    settings: {},
  },
  networks: {
    passetHub: {
      polkavm: true, // REQUIRED for Polkadot
      url: "https://services.polkadothub-rpc.com/testnet",
      accounts: [process.env.PRIVATE_KEY],
    },
  },
};
```

## Development Workflow

### Essential Commands

```bash
# Development
bun install                  # Install dependencies
bun dev                      # Start dev server (port 5173)
bun run build               # Build for production
bun run reinstall           # Clean reinstall

# Contracts
cd contracts
bun run compile             # Compile contracts
bun run test                # Run contract tests
bun run deploy-contract     # Deploy to Paseo + generate types

# Testing
bun run test:e2e           # Run e2e tests (Playwright)
bun run test:api           # Test API functions
bun run test:contract      # Run contract tests
bun run test:all           # Run all tests

# Code Generation
bun run generate           # Generate Wagmi types from contract
```

### Private Key Setup

**IMPORTANT**: Never commit private keys
```bash
# In contracts/.env
PRIVATE_KEY=your_private_key_without_0x_prefix
```

### Get Testnet Tokens
1. Visit [Polkadot Faucet](https://faucet.polkadot.io/)
2. Enter your wallet address
3. Receive PAS tokens for testing

## Smart Contract Architecture

### Donate.sol

**Contract Features:**
- Artist donation tracking by ID (Spotify artist ID)
- Platform fee: 1% (100 basis points)
- Artist claiming system
- Withdrawal mechanisms for artists and platform

**Key Functions:**
```solidity
// Donate to artist
donateToArtist(string memory artistId) external payable

// Artist claims identity
claimArtist(string memory artistId) external

// Artist withdraws donations
withdrawDonates(string memory artistId, address recipient) external

// Owner withdraws platform fees

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [janmichek/melodot](https://github.com/janmichek/melodot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
