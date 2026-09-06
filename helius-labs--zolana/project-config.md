---
trigger: always_on
description: `docs/spec.md` is the protocol source of truth. Do not edit it as part of
---

# Zolana Contributor Notes

## Source Of Truth

`docs/spec.md` is the protocol source of truth. Do not edit it as part of
implementation cleanup unless that is the explicit task. If code, tests, and the
spec disagree, treat the code or tests as suspect first.

## Repo Structure

program-libs
- libraries used in programs
- are published as crates

programs
- must not depend on sdk libs
- are not published as crates

program-tests
- integration tests for programs
- are not published as crates

sdk-libs
- libraries to interact with programs

sdk-tests
- integration test programs for sdks

prover
- go circuits
- go prover server
- rust prover client

## Workspace Shape

- `programs/shielded-pool`: the SPP Solana program.
- `program-libs/interface`: shared instruction data, tags, constants, and layout
  helpers.
- `program-tests`: internal test crates and test-only SBF programs.
- `sdk-libs`: externally useful Rust SDK crates.
- `cli`: local developer/operator tooling.
- `forester`: compilable forester skeleton for future nullifier-tree
  maintenance work.
- `prover`: proof client and prover server.

## Project Structure

```text
programs/shielded-pool/src/
  lib.rs               -- entrypoint
  processor.rs         -- instruction dispatch by tag
  error.rs             -- program error conversions
  instructions/
    loader.rs          -- account loading and shared account validation
    <instruction>/
      mod.rs           -- wires processor/verify/init helpers together
      processor.rs     -- signer checks, parsing handoff, business flow
      verify.rs        -- account/data verification for that instruction
      init.rs          -- account initialization helpers when needed

program-libs/interface/src/
  lib.rs               -- canonical program ids and public modules
  instruction/
    tag.rs             -- first-byte instruction tags
    builders/          -- client instruction builders
    instruction_data/  -- Borsh or fixed-layout instruction data structs
  state/               -- client-visible account headers and discriminators
  verifying_keys/      -- verifier constants when proof paths need them

program-tests/
  shielded-pool/       -- internal litesvm/localnet tests

sdk-libs/
  keypair/             -- shielded key material and hashes
  program-test/        -- reusable local test/indexer harness
  transaction/         -- wallet, UTXO, encryption, and transaction logic

cli/                   -- root Zolana developer/operator CLI
forester/              -- forester skeleton
prover/                -- Rust prover client and Go prover server
xtask/                 -- workspace maintenance tools
```

## Common Commands

Use `just` recipes for normal workflows:

```bash
just check-all
just test-hermetic
just test-shielded-pool
just test-sdk-libs
just test-programs
just test-cli
just clippy
```

`just test-hermetic` is the whole suite that needs nothing running, and CI runs
these same suites on every push. `just test-all` adds the prover-backed suites,
and the validator suites stay in their own recipes. Those tiers sit behind the
`proofs` and `localnet` Cargo features, so a plain `cargo test -p <crate>` never
starts a prover.

Program tests that load real SBF binaries need the local builds:

```bash
just build-programs
```

### Per-clone port isolation (`ZOLANA_PORT_OFFSET`)

Localnet/prover-backed tests bind fixed ports (RPC 8899, photon 8784, prover
3001), so two clones running them at once contend. To isolate a clone, set a
single offset in a local `.env` (gitignored, auto-loaded by `just` via `set
dotenv-load`); the justfile shifts every service port by it and derives the
matching URLs:

```bash
cp .env.example .env        # then set e.g. ZOLANA_PORT_OFFSET=100
```

Offset 100 -> RPC 8999, photon 8884, prover 3101. Use 0 / 100 / 200 / ... per
clone (stay below ~900). The justfile exports `ZOLANA_PROVER_URL`, and the
tests read `ZOLANA_LOCALNET_URL` / `ZOLANA_INDEXER_URL` / `ZOLANA_PROVER_URL`,
so the offset flows into every `just test-*` recipe. Individual
`ZOLANA_LOCALNET_RPC_PORT` / `ZOLANA_LOCALNET_PHOTON_PORT` /
`ZOLANA_LOCALNET_PROVER_PORT` (and the URL vars) still override the derived
value when set explicitly.

`ZOLANA_PROVER_URL` is the single source of truth for the prover: the client
connects there and `spawn_prover()` starts the spawned server on that URL's
port. Running `cargo test` directly (not via `just`) does not auto-load `.env`
-- export the vars yourself (`set -a; source .env; set +a`) or use `direnv`.

## Code Style

- Keep protocol math in one canonical implementation and reuse it from tests.
- Keep public SDK surface deliberate; test-only helpers belong under
  `program-tests` unless they are useful to external developers.
- Avoid compatibility shims for removed Light/legacy surfaces.
- Prefer small, explicit helpers over broad abstractions.
- Comments should explain invariants, security constraints, or non-obvious
  layout decisions. Remove comments that only narrate the code.
- Never add `#[allow(clippy::too_many_arguments)]`. Restructure with the
  method-patterns skill instead: an operation struct holding all inputs plus a
  consuming method that takes only the signer/context (e.g.
  `EscrowSettle { .. }.sign(keypair, assets)`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [helius-labs/zolana](https://github.com/helius-labs/zolana) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
