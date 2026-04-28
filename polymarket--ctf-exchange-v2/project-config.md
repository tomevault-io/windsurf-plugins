---
trigger: always_on
description: Polymarket's CTF Exchange v2 — an EIP-712 order-matching exchange for Conditional Tokens Framework (CTF) assets. Operators match signed off-chain orders on-chain with gas-optimized settlement paths. Built with Foundry and Solady.
---

# CLAUDE.md

## Project Overview

Polymarket's CTF Exchange v2 — an EIP-712 order-matching exchange for Conditional Tokens Framework (CTF) assets. Operators match signed off-chain orders on-chain with gas-optimized settlement paths. Built with Foundry and Solady.

Core concepts:
- **CTFExchange**: Main entry point. Mixin-based architecture combining Auth, Fees, Signatures, Trading, Pausable, and ERC1155TokenReceiver. Operator-gated order matching with admin controls.
- **Order Matching**: Three settlement paths — COMPLEMENTARY (BUY vs SELL, direct P2P transfers), MINT (both BUY, split collateral into outcome tokens), MERGE (both SELL, merge outcome tokens back to collateral).
- **Adapters**: CtfCollateralAdapter bridges CollateralToken ↔ ConditionalTokens (split/merge/redeem). NegRiskCtfCollateralAdapter extends it for NegRisk markets with position conversion.
- **CollateralToken**: UUPS-upgradeable ERC20 (pUSD, 6 decimals) wrapping USDC/USDCe, with permissionless onramp/offramp and EIP-712 witness-signed PermissionedRamp.

## Post-Audit Policy

This codebase has been audited. All future changes must follow these constraints:

- **Fixes only**: Only bug fixes and scoped improvements are allowed. No major refactors.
- **Minimal diffs**: Keep changes tightly scoped to the fix. Do not restructure surrounding code, rename variables, reorder functions, or "clean up" adjacent code.
- **Do NOT modify existing NatSpec** unless the fix directly changes the documented behavior.
- **Do NOT modify existing test setup** (BaseExchangeTest, TestHelper, mocks) unless the fix requires it.
- **New code requirements**: All new or modified production code must include:
  1. NatSpec documentation (see NatSpec Documentation section)
  2. Unit tests covering every new line and branch (see Testing section)
- **Gas snapshots**: If a fix changes gas usage, regenerate `.gas-snapshot` with `forge snapshot`.

## Gas Consciousness

This codebase is heavily gas-optimized — assembly-level event emission (`log4`/`log3`/`log2` in `Events.sol`), packed storage slots (`OrderStatus` packs `filled` + `remaining` into 32 bytes), COMPLEMENTARY settlement that eliminates exchange intermediary transfers, inline assembly for CREATE2 and price calculations, and batch CTF operations. Every change must respect this standard.

Before making any change, think hard about gas impact:

- **Always call out** why a change is necessary and what the trade-offs are with respect to gas. If a fix increases gas, explicitly state the cost and justify why correctness or security outweighs it.
- **Prefer gas-efficient patterns**: Use assembly where the codebase already does (events, hashing, address computation). Avoid unnecessary storage reads/writes. Batch operations where possible. Use Solady over OpenZeppelin.
- **Measure impact**: Run `forge snapshot` before and after every change and compare the `.gas-snapshot` diff. If gas increases substantially on hot paths (order matching, settlement), flag it for review.
- **Storage layout matters**: Pack related fields into single slots. Avoid adding new storage variables to frequently-read contracts without considering SLOAD cost.
- **Hot path awareness**: `matchOrders` → `_matchOrders` → `_settleComplementary` / `_settleMakerOrders` is the hot path. Any gas regression here affects every trade.

## Build & Verification Commands

```bash
forge build        # Compile
forge test         # Run all tests
forge fmt          # Format all files
forge snapshot     # Regenerate .gas-snapshot
```

**CI** (`.github/workflows/test.yml`) runs: `forge fmt --check`, `forge build --sizes`, `forge test -vvv`.

After every code change, run all four commands in order and fix any issues before considering the task complete.

## Project Structure

```
src/
├── exchange/
│   ├── CTFExchange.sol                    # Main contract — operator-gated matchOrders, preapproveOrder
│   ├── interfaces/
│   │   ├── IAuth.sol                      # Auth events/errors (IAuthEE)
│   │   ├── IAssets.sol                    # Asset address getters
│   │   ├── IConditionalTokens.sol         # CTF interface
│   │   ├── IFees.sol                      # Fee events/errors (IFeesEE)
│   │   ├── IPausable.sol                  # Pause events/errors (IPausableEE)
│   │   ├── ISignatures.sol                # Signature events/errors (ISignaturesEE)
│   │   ├── ITrading.sol                   # Trading events/errors (ITradingEE)
│   │   ├── IUserPausable.sol              # User pause events/errors (IUserPausableEE)
│   │   ├── IOutcomeTokenFactory.sol       # Outcome token factory interface
│   │   └── ISafeFactory.sol               # Gnosis Safe factory interface
│   ├── mixins/
│   │   ├── Auth.sol                       # Admin/Operator role management
│   │   ├── Assets.sol                     # Collateral, CTF, OutcomeTokenFactory addresses
│   │   ├── AssetOperations.sol            # CTF mint/merge/transfer operations
│   │   ├── ERC1155TokenReceiver.sol       # ERC1155 receive callbacks
│   │   ├── Events.sol                     # Optimized event emission (log4/log3/log2)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Polymarket) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
