---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Roxonn is a decentralized platform that integrates GitHub contributions with XDC blockchain rewards. It manages:
- GitHub OAuth authentication and repository registration
- Multi-currency reward distribution (XDC, ROXN token, USDC)
- Wallet management via Tatum API
- AI-powered development assistance with multiple model providers
- Course subscriptions with USDC payments
- Proof-of-compute node management

## Critical Development Commands

### Local Development
```bash
# Install dependencies
npm install

# Run full-stack development server
npm run dev

# Run backend server only (uses tsx)
npm run dev:server

# Build for production
npm run build

# Start production server
npm run start

# Type checking
npm run check
```

### Database Operations
```bash
# Push schema changes to database
npm run db:push

# Migrate secrets to AWS Parameter Store (if using)
npm run migrate-secrets
```

### Smart Contract Operations
```bash
# Compile contracts
npx hardhat compile

# Deploy to XDC testnet (Apothem)
npx hardhat run scripts/deploy_dual_currency_rewards.cjs --network xdcTestnet

# Deploy to XDC mainnet
npx hardhat run scripts/deploy_dual_currency_rewards.cjs --network xinfin

# Verify contract on XDCScan
npx hardhat verify --network xinfin <CONTRACT_ADDRESS>
```

### Testing
```bash
# Run tests (when implemented)
npm test

# Run reward feature tests (placeholder - tests not yet implemented)
npm run test:reward
```

## Private Repository Support

The platform supports private GitHub repositories using **GitHub App installation tokens** for access validation:

- **Pool Managers**: Can register and fund private repositories with GitHub App installed
- **Contributors**: Automatically see private repos they have GitHub access to (no OAuth upgrade needed)
- **Access Validation**: Real-time collaborator checks using GitHub App tokens (privacy-preserving)
- **No Token Storage**: Contributors' private tokens are NOT stored or requested

Implementation details in `/docs/FEATURES/PRIVATE_REPOS.md`

## Architecture Details

### Core Directory Structure
```
/server           - Express backend server
  index.ts        - Server entry point with middleware setup
  routes.ts       - Thin wrapper that delegates to modular routes
  blockchain.ts   - XDC blockchain interaction service
  walletService.ts- Wallet management & Tatum integration
  auth.ts         - GitHub OAuth & JWT authentication
  db.ts           - PostgreSQL with Drizzle ORM
  config.ts       - Configuration management (env vars + AWS SSM)
  /services       - Business logic services
  /routes         - Modular route handlers (see below)

/client          - React frontend
  /src/pages     - Application pages
  /src/components- UI components (shadcn/ui based)
  /src/hooks     - Custom React hooks
  /src/lib       - Utilities and configurations

/contracts       - Solidity smart contracts
  DualCurrencyRepoRewards.sol - Main unified rewards contract (XDC/ROXN/USDC)
  ROXNToken.sol               - ROXN ERC20/XRC20 token
  CustomForwarder.sol         - Meta-transaction forwarder
  ProofOfCompute.sol          - Compute node management
  (Legacy: RepoRewards.sol, RoxnRewards.sol, USDCRepoRewards.sol - NOT IN USE)

/shared          - Shared types and schema
  schema.ts      - Database schema & TypeScript types

/migrations      - Database migrations
  0013_add_subscriptions.sql - Latest: subscription tables
```

### Modular Routes Structure

Routes are organized in `server/routes/` with each file handling a specific domain:

```
/server/routes
  index.ts              - registerModularRoutes() - registers all route modules
  authRoutes.ts         - /api/auth/* - Authentication endpoints
  blockchainRoutes.ts   - /api/blockchain/* - Blockchain operations
  walletRoutes.ts       - /api/wallet/* - Wallet operations
  subscriptionRoutes.ts - /api/subscription/* - Subscription management
  repositoryRoutes.ts   - /api/* - Repository management
  adminRoutes.ts        - /api/admin/* - Admin operations
  nodeRoutes.ts         - /api/node/* - Compute node endpoints
  webhookRoutes.ts      - /webhook/* - GitHub App & payment webhooks
  aiRoutes.ts           - /api/vscode/* - AI completions
  miscRoutes.ts         - /health, /api/courses/*, /api/zoho/* - Misc endpoints
  referralRoutes.ts     - /api/referral/* - Referral system
  promotionalBounties.ts- /api/promotional/* - Promotional bounties
  multiCurrencyWallet.ts- /api/wallet/* - Multi-currency features
  leaderboardRoutes.ts  - /api/leaderboard/* - Leaderboard endpoints
  aiScopingAgent.ts     - /api/ai-scoping/* - AI scoping agent
```

### Active Smart Contracts

The system uses UUPS proxy pattern for upgradeability. Current active contracts:

1. **DualCurrencyRepoRewards** (Main Contract)
   - Handles XDC, ROXN, and USDC rewards in one unified contract
   - Proxy + Implementation pattern
   - Env vars: `DUAL_CURRENCY_REWARDS_CONTRACT_ADDRESS` (proxy)

2. **ROXNToken**
   - ERC20/XRC20 token implementation
   - 1 billion max supply
   - Env var: `ROXN_TOKEN_ADDRESS`

3. **CustomForwarder**
   - Meta-transactions for gasless operations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Roxonn-FutureTech/Roxonn-Platform](https://github.com/Roxonn-FutureTech/Roxonn-Platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
