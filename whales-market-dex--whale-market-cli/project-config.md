---
trigger: always_on
description: CLI for the [Whales Market](https://whales.market) pre-market trading platform. Supports multi-chain wallet management, offer creation/filling/settling, OTC trading, and order book queries across EVM, Solana, Sui, and Aptos.
---

# whale-market-cli

CLI for the [Whales Market](https://whales.market) pre-market trading platform. Supports multi-chain wallet management, offer creation/filling/settling, OTC trading, and order book queries across EVM, Solana, Sui, and Aptos.

## Commands

```bash
npm run build       # Compile TypeScript → dist/
npm run dev         # Run via ts-node (development)
npm test            # Jest
npm start           # Run compiled output
```

No lint command configured.

## Project Layout

```
src/
├── index.ts                      # CLI entry point — registers all commands, suppresses punycode warnings
├── types.ts                      # Shared types: Token, Offer, Order, ApiResponse, OutputFormat
├── api.ts                        # ApiClient singleton (axios, V2 endpoints, error interceptor)
├── config.ts                     # Persistent config via `conf` (wallets, RPC overrides, settings)
├── auth.ts                       # JWT/token management
├── load-env.ts                   # Loads ~/.whales-cli.env into process.env
│
├── commands/                     # One file per top-level CLI command group
│   ├── wallet.ts                 # `whales wallet` — create, import, list, use, show, remove
│   ├── trade.ts                  # `whales trade` — create-offer, fill-offer, close-offer, settle, claim-collateral
│   ├── otc.ts                    # `whales otc` — create, fill, cancel OTC positions
│   ├── offers.ts                 # `whales offers` — list, my, get, react
│   ├── orders.ts                 # `whales orders` — list, my, get, by-offer
│   ├── tokens.ts                 # `whales tokens` — list, get, search, highlight, stats
│   ├── book.ts                   # `whales book` — order book with depth option
│   ├── orderbook.ts              # `whales orderbook` — V2 aggregated order book (snapshot, positions, pairs)
│   ├── portfolio.ts              # `whales portfolio` — show, positions, balance
│   ├── referral.ts               # `whales referral` — summary, campaigns, earnings, transactions
│   ├── config.ts                 # `whales config` — get/set config + RPC subcommands
│   ├── networks.ts               # `whales networks` — list supported chains
│   ├── status.ts                 # `whales status` — API health + active wallet info
│   ├── setup.ts                  # `whales setup` — interactive first-time setup wizard
│   ├── shell.ts                  # `whales shell` — REPL mode
│   ├── completion.ts             # `whales completion` — zsh/bash shell completion
│   ├── upgrade.ts                # `whales upgrade` — self-update from npm
│   ├── help.ts                   # `whales help` — extended help docs
│   └── helpers/
│       ├── chain.ts              # Chain resolution: getChainType, is*Chain, getPreMarket, resolveRpc
│       ├── resolve.ts            # UUID → on-chain ID: resolveToken, resolveOffer, resolveOrder, resolveOtcOffer
│       └── confirm.ts            # confirmTx() — interactive confirmation prompt, respects --yes
│
├── blockchain/                   # Chain-specific contract implementations
│   ├── types.ts                  # Shared interfaces: ChainAdapter, TxResult, OfferData, DiscountData
│   ├── evm/
│   │   ├── constants.ts          # EVM_CHAINS: chain configs, default RPC URLs, explorer links
│   │   ├── signer.ts             # deriveEvmWallet(), EvmAdapter
│   │   ├── utils.ts              # parseUnits, formatUnits, encodeSettleData, isReferralNetwork, ETH_ADDRESS
│   │   ├── ex-tokens.ts          # Supported exchange tokens (USDC/USDT) per EVM chain
│   │   └── contracts/
│   │       ├── PreMarket.ts      # EvmPreMarket — create/fill/close/settle/claim via ethers.js
│   │       ├── OtcPreMarket.ts   # EvmOtcPreMarket — OTC operations
│   │       └── abis/             # ABI JSON for PreMarket, PreMarketRef, OtcPreMarket, ERC20
│   ├── solana/
│   │   ├── constants.ts          # SOLANA_RPC URLs, Anchor program IDs
│   │   ├── signer.ts             # deriveSolanaKeypair(), SolanaAdapter
│   │   ├── utils.ts              # Solana-specific utilities
│   │   └── programs/
│   │       ├── PreMarket.ts      # SolanaPreMarket — Anchor-based contract calls
│   │       ├── OtcPreMarket.ts   # SolanaOtcPreMarket
│   │       └── idl/              # Anchor IDL definitions (pre_market.ts, otc_pre_market.ts)
│   ├── sui/
│   │   ├── constants.ts          # SUI_RPC URLs, Move package IDs
│   │   ├── signer.ts             # deriveSuiKeypair(), SuiAdapter
│   │   └── contracts/
│   │       ├── PreMarket.ts      # SuiPreMarket — Move contract calls via @mysten/sui
│   │       └── OtcPreMarket.ts   # Stub — throws (not yet implemented)
│   └── aptos/
│       ├── constants.ts          # APTOS_RPC URLs, Move module addresses
│       ├── signer.ts             # deriveAptosAccount(), AptosAdapter
│       └── contracts/
│           ├── PreMarket.ts      # AptosPreMarket — Move contract calls via @aptos-labs/ts-sdk
│           └── OtcPreMarket.ts   # Stub — throws (not yet implemented)
│
├── output/                       # Output formatting layer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Whales-Market-Dex/whale-market-cli](https://github.com/Whales-Market-Dex/whale-market-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
