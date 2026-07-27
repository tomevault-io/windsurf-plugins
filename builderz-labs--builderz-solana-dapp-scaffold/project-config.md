---
trigger: always_on
description: This file configures AI coding assistants for Solana program development. Follow these rules to produce secure, optimized, and correct code.
---

# Solana Development Rules

This file configures AI coding assistants for Solana program development. Follow these rules to produce secure, optimized, and correct code.

## Modern Tooling (2025)

**USE THESE - NOT OUTDATED ALTERNATIVES:**

| Category | Modern (Use This) | Outdated (Avoid) |
| -------- | ----------------- | ---------------- |
| Program SDK | `pinocchio`, `solana-program` 2.x | `solana-program` 1.x |
| Framework | Anchor 0.32+, Steel | Anchor < 0.30 |
| Unit Testing | Mollusk, LiteSVM | `solana-program-test` alone |
| Local Dev | Surfpool | `solana-test-validator` alone |
| Client SDK | `@solana/kit` (web3.js 2.0) | `@solana/web3.js` 1.x |
| Fuzz Testing | Trident | None |

## Stack Configuration

Uncomment and configure based on your project:

```
# Framework: anchor | native | pinocchio | steel
# Client: typescript (@solana/kit) | rust | python
# Testing: mollusk | litesvm | trident | bankrun
# Local: surfpool | solana-test-validator
```

---

## Critical Rules

### NEVER

1. **NEVER use deprecated crates:**

   - `solana-program` < 2.0 → use `solana-program` 2.x or `pinocchio`
   - `solana-sdk` < 2.0 → use `solana-sdk` 2.x
   - `spl-token` < 5.0 → use `spl-token` 5.x or `spl-token-2022`
   - `@solana/web3.js` 1.x → use `@solana/kit` (web3.js 2.0)

2. **NEVER deploy to mainnet without explicit user confirmation** - always ask first

3. **NEVER use unchecked arithmetic:**

   ```rust
   // WRONG
   let total = amount_a + amount_b;

   // CORRECT
   let total = amount_a.checked_add(amount_b).ok_or(ErrorCode::Overflow)?;
   ```

4. **NEVER skip account validation** - every account must be validated for owner, signer status, and PDA derivation

5. **NEVER hardcode keypairs, private keys, or RPC endpoints**

6. **NEVER use `unwrap()` in on-chain program code** - use proper error handling

7. **NEVER assume account data layout** - always deserialize explicitly

### ALWAYS

1. **ALWAYS verify account ownership before reading/writing:**

   ```rust
   if *account.owner != expected_program_id {
       return Err(ProgramError::IncorrectProgramId);
   }
   ```

2. **ALWAYS use canonical bump seeds for PDAs** - store and reuse the bump

3. **ALWAYS set explicit compute unit limits in transactions**

4. **ALWAYS validate all accounts passed to instructions**

5. **ALWAYS calculate rent-exempt minimum for account creation**

6. **ALWAYS simulate transactions before sending to mainnet**

---

## Framework Selection

| Scenario                 | Recommendation   | Reason                            |
| ------------------------ | ---------------- | --------------------------------- |
| Rapid prototyping        | Anchor           | Auto-generated IDL, better DX     |
| Team collaboration       | Anchor           | Standardized patterns             |
| CU optimization critical | Pinocchio        | 80-95% CU reduction               |
| Maximum control needed   | Pinocchio/Native | Zero-copy, no abstractions        |
| Minimal binary size      | Pinocchio        | Smallest footprint                |
| Native + better DX       | Steel            | Balance of control and ergonomics |

### When to Optimize

Start with Anchor. Optimize to Pinocchio/Native when:

- Transaction costs become significant at scale
- CU limits are being hit
- Binary size affects deployment costs
- Maximum throughput is required

---

## Token Program Selection

| Feature Needed              | Program          | Notes                              |
| --------------------------- | ---------------- | ---------------------------------- |
| Simple fungible token       | SPL Token        | Classic, widely supported          |
| Transfer fees               | Token-2022       | Issuer fee on transfers            |
| Confidential transfers      | Token-2022       | Encrypted balances/amounts         |
| Transfer hooks              | Token-2022       | Custom logic on transfers          |
| Permanent delegate          | Token-2022       | Compliance/seizure capability      |
| Non-transferable tokens     | Token-2022       | Soulbound tokens                   |
| Interest-bearing tokens     | Token-2022       | Auto-accruing value                |

### Token-2022 Critical Rules

```rust
// WRONG - transfer() fails with transfer fees
token::transfer(cpi_ctx, amount)?;

// CORRECT - use transfer_checked for Token-2022
token::transfer_checked(cpi_ctx, amount, decimals)?;
```

**Compatibility:** Some wallets/dApps don't support all extensions. Test on devnet with target integrators.

---

## Program Patterns

### Account Validation Order

Always validate in this order:

1. Account ownership
2. Signer status
3. PDA derivation
4. Data constraints

### Anchor Pattern

```rust
#[derive(Accounts)]
pub struct Transfer<'info> {
    #[account(
        mut,
        has_one = authority,
        constraint = source.amount >= amount @ ErrorCode::InsufficientFunds
    )]
    pub source: Account<'info, TokenAccount>,

    #[account(mut)]
    pub destination: Account<'info, TokenAccount>,

    pub authority: Signer<'info>,

    #[account(
        seeds = [b"vault", source.key().as_ref()],
        bump = vault.bump,
    )]
    pub vault: Account<'info, Vault>,
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [builderz-labs/builderz-solana-dapp-scaffold](https://github.com/builderz-labs/builderz-solana-dapp-scaffold) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
