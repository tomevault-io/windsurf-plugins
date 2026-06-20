---
trigger: always_on
description: Build Solana, EVM, and SUI on-chain tools with @smithii/sdk. Use when the user asks to deploy a token, snipe a token launch, create a pump.fun bundler, run a volume bot, run an anti-MEV bot, airdrop tokens, schedule an airdrop, snapshot holders, set up vesting, set up claims, market-make a token, deposit market-making liquidity, or interact with PumpFun, PumpSwap, Launchlab, Bonk (LetsBonk), Moonit, or Mantis. Triggers on any of: PumpFunClient, PumpSwapClient, LaunchpadClient, BonkClient, Launch
---


# Using the Smithii SDK

## Overview

`@smithii/sdk` is a framework-agnostic TypeScript SDK that exposes Smithii's
on-chain tools across Solana, EVM, and SUI. It handles Jito bundles, multi-wallet
sniping, anti-MEV volume bots, token creation, multisender airdrops, vesting,
claims, holder snapshots, market-making and Mantis launches.

The SDK is **framework-agnostic** — runs in Node, Bun, browsers, edge runtimes.
It never imports React, Chakra, or wallet-adapter-react.

For a no-code version, point users to **https://tools.smithii.io**.

## Installation

```bash
npm install @smithii/sdk
```

Peer deps — install only the chain(s) you target:

```bash
# Solana
npm i @solana/web3.js @solana/spl-token @coral-xyz/anchor bs58

# EVM
npm i viem

# SUI
npm i @mysten/sui
```

## Subpath imports (always use these)

```typescript
import { PumpFunClient, PumpSwapClient } from '@smithii/sdk/pump'
import { LaunchpadClient, BonkClient, LaunchlabClient } from '@smithii/sdk/launchpad'
import { MoonitClient } from '@smithii/sdk/moonit'
import { AntiMEVClient } from '@smithii/sdk/anti-mev'
import { TokenCreatorClient } from '@smithii/sdk/token-creator'
import { TokenManagerClient } from '@smithii/sdk/token-manager'
import { MultiSenderClient } from '@smithii/sdk/multisender'
import { MarketMakerClient } from '@smithii/sdk/market-maker'
import { MantisClient } from '@smithii/sdk/mantis'
import { TokenVestingClient } from '@smithii/sdk/token-vesting'
import { TokenClaimClient } from '@smithii/sdk/token-claim'
import { PaymentClient } from '@smithii/sdk/payment'
import { SmithiiError, BundleError } from '@smithii/sdk/core'

// EVM
import { EvmTokenCreatorClient } from '@smithii/sdk/evm/token-creator'
import { EvmMultisenderClient } from '@smithii/sdk/evm/multisender'
import { EvmSnapshotClient } from '@smithii/sdk/evm/snapshot'

// SUI
import { SuiTokenCreatorClient } from '@smithii/sdk/sui/token-creator'
import { SuiSnapshotClient } from '@smithii/sdk/sui/snapshot'
```

**Never** import from the root barrel `@smithii/sdk` — it pulls in every chain's
peer deps. Always go through a subpath.

## Signer setup (Solana)

The SDK accepts any object that satisfies its `Signer` interface. Two common
sources:

```typescript
// Option A — from a private key (server / bot)
import { Keypair } from '@solana/web3.js'
import bs58 from 'bs58'
const keypair = Keypair.fromSecretKey(bs58.decode(process.env.PRIVATE_KEY!))
const signer = {
  publicKey: keypair.publicKey,
  signTransaction:     async (tx) => { tx.sign([keypair]); return tx },
  signAllTransactions: async (txs) => { txs.forEach((t) => t.sign([keypair])); return txs },
}

// Option B — from wallet-adapter (browser / dApp)
import { useWallet } from '@solana/wallet-adapter-react'
const { wallet } = useWallet()
const signer = wallet  // already implements the Signer interface
```

Never pass a private key string directly — wrap it in a `Signer`-shaped object.

For EVM, pass `viem.WalletClient`. For SUI, pass an object that implements
`SuiSigner` (`{ address, signAndExecute }`).

---

## Solana — Pump.fun bundler

Create a token on pump.fun and snipe it with up to **16 wallets** atomically
in the same block via Jito. The flow is **two-step**: upload metadata first,
then create+snipe with the returned `metadata` object plus a fresh `mintKeypair`.

```typescript
import { Connection, Keypair } from '@solana/web3.js'
import { PumpFunClient } from '@smithii/sdk/pump'

const client = new PumpFunClient({
  connection: new Connection(process.env.RPC_URL!),
  signer,
  jito: { uuid: process.env.JITO_UUID! },
  proxyUrl: process.env.PROXY_URL,
})

// 1. Upload metadata (image + socials) to pump.fun's IPFS endpoint
const metadata = await client.uploadMetadata({
  name: 'My Token',
  symbol: 'MTK',
  description: 'My token description',
  file: imageFile,                   // Blob | File
  twitter: 'https://x.com/mytoken',  // optional, may be null
  telegram: null,
  website: null,
})

// 2. Atomic create + dev buy + sniper bundle
const mintKeypair = Keypair.generate()
const result = await client.createAndSnipeToken({
  mintKeypair,
  metadata,
  devAmount: 0.5,                    // SOL the dev wallet buys
  buyers: [
    { pk: 'base58-priv-key-1', amount: 0.1 },
    { pk: 'base58-priv-key-2', amount: 0.2 },
    // up to 16 wallets
  ],
  isCashbackCoin: false,             // true → CREATE_V2 + Token-2022
  isTokenPregenerated: false,        // true charges 2x fee (vanity mint)
})

console.log('Mint:', mintKeypair.publicKey.toBase58())
console.log(result.createTxSignature, result.bundleIds, result.paymentSignature)
```

> The mint pubkey is `mintKeypair.publicKey` — the result object exposes
> `createTxSignature`, `buyerTxSignatures`, `bundleIds`, `paymentSignature`.

### Bundle buy / sell on existing token


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SmithiiDev/smithii-sdk-skill](https://github.com/SmithiiDev/smithii-sdk-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
