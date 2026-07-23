---
trigger: always_on
description: For Hydration protocol-level context (architecture, products, tokenomics, Omnipool mechanics), fetch the central context index via WebFetch:
---

# CLAUDE.md — hydration-node

## Protocol context

For Hydration protocol-level context (architecture, products, tokenomics, Omnipool mechanics), fetch the central context index via WebFetch:
`https://raw.githubusercontent.com/galacticcouncil/hydration/main/CLAUDE.md`

It lists available reference documents and their raw GitHub URLs.

## Shared AI skills

Repo-local AI skills live in `ai_skills/` so they can be used by multiple coding
agents, not only Claude.

When the user invokes a skill by name, or the task clearly matches a skill
description, load the corresponding `ai_skills/<skill-name>/SKILL.md` file and
follow its instructions. Resolve any relative paths in a skill from that skill's
directory.

Available shared skills:
- `hydration_cl0wdit` - security audit workflow for Substrate runtime and pallet
  code.
- `circuit-breaker-incident` - investigate snakewatch lockdown alerts (XCM
  deposit fuse, trade/liquidity limits).

## Operator runbooks & docs

- `scripts/mint-limit/README.md` — generate TC proposals to set XCM mint limits and lift circuit breaker lockdowns
- `scripts/dca-monitor/README.md` — verify DCA fixes on a Chopsticks fork before deploying a runtime upgrade
- `scripts/onchain-routes/README.md` — generate TC proposals to register/update on-chain router routes
- `integration-tests/README.md` — debug prod issues via scraper snapshots + integration tests

## Project overview

Substrate-based parachain (Polkadot ecosystem) implementing DeFi protocols — DEX (Omnipool, Stableswap, XYK, LBP), DCA, OTC, bonds, staking, governance, and EVM compatibility.

**Repo:** `galacticcouncil/hydration-node`
**Runtime:** `runtime/hydradx/` — current version 399.0.0
**Pallets:** 42+ custom pallets in `pallets/`
**Toolchain:** Rust 1.84.1, target `wasm32-unknown-unknown`

## Build & test

```sh
make build           # release build
make test            # cargo test --locked
make test-release    # cargo test --release --locked
make clippy          # clippy with -D warnings (RUSTFLAGS)
make format          # cargo fmt
make build-benchmarks  # build with runtime-benchmarks feature
make test-benchmarks   # test with runtime-benchmarks feature
```

Single pallet test: `cargo test -p pallet-omnipool --locked`

All cargo commands use `--config net.git-fetch-with-cli=true` (see Makefile).

## Test Naming Convention

Use BDD-style "should-when" naming for all tests. The test name should read as a specification of behavior.

**Format:** `<subject>_should_<expected_outcome>_when_<condition>`

**Examples:**
- `transfer_should_fail_when_balance_is_insufficient`
- `route_suggester_should_return_shortest_path_when_multiple_routes_exist`
- `sell_should_succeed_when_slippage_within_limit`
- `add_liquidity_should_fail_when_pool_is_frozen`

**Rules:**
- Use `snake_case` (Rust convention).
- For success cases, use `should_<outcome>_when_<condition>` (omit "succeed" if the outcome is descriptive enough).
- For failure cases, prefer `should_fail_when_<condition>` and assert on the specific error.
- The `<subject>` is typically the function/extrinsic under test.
- Avoid generic names like `test_1`, `it_works`, or `basic_test`.
- One behavior per test — if you need "and" in the name, split it into two tests.

## Extrinsic documentation

Every public extrinsic in `#[pallet::call]` blocks must have a rustdoc comment that follows
this standard structure. See `pallets/omnipool/src/lib.rs` and `pallets/stableswap/src/lib.rs`
for canonical examples.

**Required sections (in order):**

1. **Description** — one-line summary, then any longer explanation as additional paragraphs.
   Cover what the extrinsic does, important preconditions, and notable side effects (NFT
   minting, hooks, tradability flags, error conditions worth highlighting).
2. **Parameters** — a `Parameters:` block listing every argument as `` - `name`: description ``.
   Include `origin` when its required type is non-trivial (e.g. `T::AuthorityOrigin`).
3. **Emitted events** — a final line of the form `` Emits `EventName` event when successful. ``
   If multiple events are emitted, list each on its own line.

**Format:**

```rust
/// <One-line summary of what the extrinsic does.>
///
/// <Optional longer explanation: preconditions, side effects, error conditions,
/// hook invocations, tradability flags, etc. Use multiple paragraphs as needed.>
///
/// Parameters:
/// - `origin`: <only if origin type is non-trivial, e.g. Must be T::AuthorityOrigin>
/// - `param_a`: <what it represents and any constraints>
/// - `param_b`: <what it represents and any constraints>
///
/// Emits `SomethingHappened` event when successful.
///
#[pallet::call_index(N)]
#[pallet::weight(...)]
#[transactional]
pub fn my_extrinsic(...) -> DispatchResult { ... }
```

**Rules:**
- Use `///` doc comments (rustdoc), not `//` line comments.
- Blank `///` lines separate paragraphs and the three sections.
- Wrap identifiers, types, and values in backticks (e.g. `` `asset_id` ``, `` `T::AuthorityOrigin` ``).
- Phrase the emitted-events line consistently: `` Emits `X` event when successful. ``
- If the extrinsic delegates to another (e.g. `add_liquidity` → `add_liquidity_with_limit`),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [galacticcouncil/hydration-node](https://github.com/galacticcouncil/hydration-node) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
