---
trigger: always_on
description: Integrate Solana SPL token payments (USDC/USDT) into the Toppio payment provider system. Use this skill when implementing the Solana payment provider, handling SPL token transfers, building sweep logic for Solana, or debugging Solana-related payment issues.
---


This skill guides implementation of a Solana SPL token payment provider for Toppio's existing chain-agnostic `PaymentProvider` interface. It covers wallet generation, balance checking, transfer detection, and sweeping — all adapted to Solana's unique account model.

## Solana vs EVM — Key Differences

Solana is NOT EVM-compatible. Do not use ethers.js or EVM patterns. Key differences:

| Concept | EVM (BSC/Polygon) | Solana |
|---------|-------------------|--------|
| Library | ethers.js v6 | @solana/web3.js + @solana/spl-token |
| Keypair | Random wallet (secp256k1) | Ed25519 keypair |
| Token balances | ERC-20 contract.balanceOf() | Associated Token Account (ATA) |
| Token transfer | contract.transfer() | createTransferInstruction() |
| Gas token | BNB / POL | SOL |
| Gas cost | ~$0.01-0.05 | ~$0.001 (base) + ~$0.40 (ATA creation) |
| Finality | ~3-15s | ~400ms (optimistic), ~30s (confirmed) |
| Block scanning | Transfer events via getLogs | getSignaturesForAddress + getParsedTransaction |

## Token Addresses (Mainnet)

```typescript
// USDC — Token Program (not Token-2022)
const USDC_MINT = 'EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v';

// USDT — Token Program (not Token-2022)
const USDT_MINT = 'Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB';

// Token Program address (both USDC and USDT use this, NOT Token-2022)
const TOKEN_PROGRAM_ID = 'TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA';
```

**CRITICAL**: USDC and USDT on Solana both use the original Token Program, NOT Token-2022. Using the wrong program produces invalid ATAs and failed transfers.

## Dependencies

```bash
npm install @solana/web3.js@1 @solana/spl-token bs58
```

Use `@solana/web3.js` v1 (stable). v2 is a full rewrite with different APIs. `bs58` is needed for encoding/decoding Solana keypairs (not built into web3.js).

## Provider Implementation

### File: `providers/payment/solana.ts`

Implement the `PaymentProvider` interface from `providers/payment/types.ts`:

```typescript
interface PaymentProvider {
  readonly chain: string;           // 'Solana'
  readonly chainId: string;         // 'solana'
  readonly token: string;           // 'USDC/USDT'
  readonly decimals: number;        // 6 (both USDC and USDT on Solana are 6 decimals)
  readonly gasToken: string;        // 'SOL'

  generateDepositAddress(): Promise<DepositAddress>;
  checkBalance(address: string): Promise<BalanceResult>;
  getIncomingTransfers(address: string, fromBlock?: number): Promise<TransferInfo[]>;
  sweep(fromPrivateKey: string, toAddress: string): Promise<SweepResult>;
  getExplorerUrl(txHash: string): string;
  getAddressExplorerUrl(address: string): string;
  isValidAddress(address: string): boolean;
  getMasterGasBalance(): Promise<string>;
}
```

### Wallet Generation

```typescript
import { Keypair } from '@solana/web3.js';
import bs58 from 'bs58';

async generateDepositAddress(): Promise<DepositAddress> {
  const keypair = Keypair.generate();
  return {
    address: keypair.publicKey.toBase58(),
    // Store secret key as base58 string (compatible with existing encrypt/decrypt)
    privateKey: bs58.encode(keypair.secretKey),
  };
}
```

Note: Solana secret keys are 64 bytes (includes the public key). Store the full secretKey, not just the seed.

### Address Validation

```typescript
import { PublicKey } from '@solana/web3.js';

isValidAddress(address: string): boolean {
  try {
    new PublicKey(address);
    return true;
  } catch {
    return false;
  }
}
```

### Balance Checking

```typescript
import { Connection, PublicKey } from '@solana/web3.js';
import { getAssociatedTokenAddress, getAccount, TokenAccountNotFoundError } from '@solana/spl-token';

async checkBalance(address: string): Promise<BalanceResult> {
  const connection = new Connection(this.rpcUrl);
  const owner = new PublicKey(address);
  let total = 0n;

  for (const mintStr of [USDC_MINT, USDT_MINT]) {
    const mint = new PublicKey(mintStr);
    const ata = await getAssociatedTokenAddress(mint, owner);
    try {
      const account = await getAccount(connection, ata);
      total += account.amount; // bigint, already in smallest units
    } catch (e) {
      if (e instanceof TokenAccountNotFoundError) continue; // ATA doesn't exist yet
      throw e;
    }
  }

  const balance = Number(total) / 10 ** this.decimals;
  return { balance, raw: total.toString() };
}
```

**Important**: ATAs may not exist for new deposit wallets. This is normal — the ATA gets created when the sender transfers tokens. Most wallets and dApps handle ATA creation automatically.

### Incoming Transfer Detection

Solana doesn't have event logs like EVM. Use signature history + parsed transactions:

```typescript
import { Connection, PublicKey, ParsedTransactionWithMeta } from '@solana/web3.js';

async getIncomingTransfers(address: string): Promise<TransferInfo[]> {
  const connection = new Connection(this.rpcUrl);
  const owner = new PublicKey(address);
  const transfers: TransferInfo[] = [];

  for (const mintStr of [USDC_MINT, USDT_MINT]) {
    const mint = new PublicKey(mintStr);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sepivip/solana-payments-skill](https://github.com/sepivip/solana-payments-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
