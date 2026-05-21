---
trigger: always_on
description: Library of audited Soroban smart-contract building blocks for the Stellar
---

# OpenZeppelin Stellar Contracts — Agent Guide

Library of audited Soroban smart-contract building blocks for the Stellar
network. Contracts target `wasm32v1-none`; library crates are `no_std`.

## Repository map

```
packages/
├── access/          # access_control, ownable, role_transfer
├── accounts/        # smart-account policies (multisig, thresholds)
├── contract-utils/  # pausable, upgradeable, math, crypto, merkle_distributor
├── fee-abstraction/ # fee forwarder
├── governance/      # governor, timelock, votes
├── macros/          # only_owner / only_role / when_paused, etc.
├── test-utils/      # event-assertion helper crate
├── tokens/          # fungible + non_fungible + rwa + vault
└── zk-email/        # zk-email auth primitives
examples/            # one example crate per feature; cdylib only
audits/              # security audit reports
Architecture.md      # in-depth design doc — read this first for context
CONTRIBUTING.md      # workflow, PR rules, AI-usage policy
```

Each library package follows the same shape:

```
<package>/src/<module>/
├── mod.rs       # docstring, trait (#[contracttrait]), errors, constants, events
├── storage.rs   # #[contracttype] storage keys + free functions
└── test.rs      # #[cfg(test)] mod test;
```

## Commands you'll run

```bash
# Format (NIGHTLY required — rustfmt.toml uses unstable_features)
cargo +nightly fmt --all -- --check

# Lint (warnings are errors in CI)
cargo clippy --release --locked --all-targets -- -D warnings

# Test the workspace
cargo test

# WASM release build (per-package; see CI note in .github/workflows/generic.yml)
cargo build --target wasm32v1-none --release --package <name>

# Coverage (CI threshold: 90% line coverage)
cargo llvm-cov --workspace --fail-under-lines 90
```

When iterating, prefer `--package <name>` to avoid rebuilding the whole
workspace.

## Conventions you must follow

These are non-obvious rules where AI-generated drafts most often go wrong.
For the full checklist see `.claude/skills/code-quality.md`.

### Storage TTL

- Three tiers: `instance` (singletons), `persistent` (long-lived per-key),
  `temporary` (time-bounded — allowances, pending transfers).
- **Library-owned `persistent`/`temporary` reads extend TTL; writes do not.**
  Canonical pattern:
  ```rust
  if let Some(value) = e.storage().persistent().get::<_, T>(&key) {
      e.storage().persistent().extend_ttl(&key, FOO_TTL_THRESHOLD, FOO_EXTEND_AMOUNT);
      value
  } else {
      Default::default()
  }
  ```
  Argument order is always `(&key, TTL_THRESHOLD, EXTEND_AMOUNT)`.
- `instance` TTL is the **contract developer's** responsibility, not the
  library's. Libraries expose `INSTANCE_TTL_THRESHOLD` and
  `INSTANCE_EXTEND_AMOUNT` as defaults but never call
  `instance().extend_ttl()` themselves.
- Constants follow the trio:
  ```rust
  const DAY_IN_LEDGERS: u32 = 17280;
  pub const FOO_EXTEND_AMOUNT: u32 = N * DAY_IN_LEDGERS;
  pub const FOO_TTL_THRESHOLD: u32 = FOO_EXTEND_AMOUNT - DAY_IN_LEDGERS;
  ```

### Authorization

- `panic_with_error!(e, ErrorEnum::Variant)` — never bare `panic!` /
  `unwrap()` in non-test code. (`expect("...")` is fine when the message
  explains *why* the value is guaranteed to exist.)
- **Never call `require_auth()` twice on the same address in one
  invocation** — Soroban panics. This is why `#[has_role]` /
  `#[has_any_role]` exist alongside `#[only_role]` / `#[only_any_role]`:
  - `only_*` injects `require_auth()`. Use when the body doesn't already
    require auth.
  - `has_*` does NOT inject `require_auth()`. Use when the body (or the
    `Base::` helper it delegates to) already does.
- High-level fn does auth + emits event. Low-level `_no_auth` sibling does
  neither and accepts `caller: &Address` purely for the event. Don't mix
  the two halves.
- Functions that intentionally skip auth (e.g. `Base::mint`,
  `pausable::pause`, `set_admin`, `set_owner`, `set_metadata`) MUST carry a
  `# Security Warning` doc block explaining the missing check.

### Trait / contract-type pattern

Token-style traits use an associated type to enforce mutually-exclusive
extensions at compile time:

```rust
#[contracttrait]
pub trait FungibleToken {
    type ContractType: ContractOverrides;
    fn transfer(e: &Env, from: Address, to: MuxedAddress, amount: i128) {
        Self::ContractType::transfer(e, &from, &to, amount);
    }
}
```

In contract `impl` blocks:

- `#[contractimpl(contracttrait)]` on `impl <Trait> for Contract` — picks up
  default method bodies from the `#[contracttrait]`.
- `#[contractimpl]` (no arg) on the contract's inherent `impl` block (the
  one with `__constructor` and contract-specific helpers).

Reversing the two is a common mistake.

### Errors, events, sections

- Errors: `#[contracterror] #[derive(Copy, Clone, Debug, Eq, PartialEq,
  PartialOrd, Ord)] #[repr(u32)]`, PascalCase variants. Stay within the
  package's existing numeric range (pausable 1000s, fungible 100s,
  access-control 2000s, ownable 2100s, …) — don't invent new ranges.
- Events: `#[contractevent]` struct (PascalCase, past-tense or noun) +
  snake_case `pub fn emit_<event>(e: &Env, ...)` helper that
  `.publish(e)`s. Don't call `.publish(e)` directly from `storage.rs` —
  funnel through the helper in `mod.rs`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenZeppelin/stellar-contracts](https://github.com/OpenZeppelin/stellar-contracts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
