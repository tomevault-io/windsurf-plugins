---
trigger: always_on
description: Cargo subcommand and toolchain for building Rust smart contracts targeting PolkaVM (used by Polkadot's `pallet-revive`). Scaffolds projects from Solidity `.sol` interfaces, generates ABI encoding/decoding, and compiles to `.polkavm` bytecode.
---

# cargo-pvm-contract

Cargo subcommand and toolchain for building Rust smart contracts targeting PolkaVM (used by Polkadot's `pallet-revive`). Scaffolds projects from Solidity `.sol` interfaces, generates ABI encoding/decoding, and compiles to `.polkavm` bytecode.

## Crate Overview

| Crate | Description |
|-------|-------------|
| `cargo-pvm-contract` | CLI tool — scaffolds contract projects from `.sol` files |
| `cargo-pvm-contract-builder` | Build library — links PolkaVM bytecode and generates ABI JSON (used by CLI and optional `build.rs`) |
| `pvm-contract-sdk` | Primary user-facing SDK crate — re-exports macros, types, and polkavm-derive for contract development |
| `pvm-contract-core` | Core structures for the PVM smart contracts SDK |
| `pvm-contract-macros` | Proc macros — `#[contract]`, `#[method]`, `#[selector]`, `#[payable]`, `#[constructor]`, `#[fallback]`, `#[receive]`, `#[storage]`, `#[interface_id]`, `abi_import!`, `#[derive(SolType)]`, `#[derive(SolStorage)]`, `#[derive(SolError)]`, `#[derive(SolEvent)]` |
| `pvm-contract-types` | ABI encoding/decoding traits (`SolEncode`, `SolDecode`), error trait (`SolError`) — `no_std` compatible |
| `pvm-storage` | Typed storage helpers — `Lazy<T>`, `Mapping<K, V>`, Solidity-compatible slot layout |
| `pvm-contract-builder-dsl` | Builder-pattern DSL for contracts without proc macros |
| `cargo-pvm-contract-extrinsics` | Library defining extrinsics for PVM smart contracts on pallet-revive |
| `pvm-bump-allocator` | Simple bump allocator for PVM smart contracts (backs `allocator = "bump"`) |
| `pvm-contract-benchmarks` | Binary size comparison tool for CI regression detection |
| `pvm-contract-e2e-tests` | End-to-end + integration test harness |
| `pvm-solc-differential` | Differential tests of on-chain storage representation vs real solc (executed on `revm`); `solc-tests` feature |

## How It Works

### End-to-End Pipeline

```
cargo pvm-contract (CLI)
    |
    v
Scaffold project from .sol interface or template
    |
    v
cargo build --release  (user runs this in the scaffolded project)
    |
    v
[build.rs] PvmBuilder::new().build()
    |
    +-- cargo build --target riscv64emac-unknown-none-polkavm -Zbuild-std=core,alloc
    |     |
    |     +-- #[contract] macro expands: dispatch + selectors + encode/decode
    |     +-- SolEncode/SolDecode trait impls handle ABI wire format
    |     +-- Output: ELF binary
    |
    +-- polkavm_linker (strip + optimize, TargetInstructionSet::ReviveV1)
    |     Output: target/{profile}/{binary}.polkavm
    |
    +-- ABI generation (parse .sol or run with --features abi-gen)
          Output: target/{profile}/{binary}.abi.json
```

### Two API Styles

**Macro API** (declarative, auto-ABI):
```rust
#[pvm_contract_sdk::contract("MyToken.sol", buffer = 256)]
mod my_token {
    pub struct MyToken;

    impl MyToken {
        #[pvm_contract_sdk::constructor]
        pub fn new(&mut self) -> Result<(), Error> { Ok(()) }

        #[pvm_contract_sdk::method]
        pub fn balance_of(&self, account: Address) -> U256 {
            self.host().get_storage(/* ... */);
            /* ... */
        }

        #[pvm_contract_sdk::fallback]
        pub fn fallback(&mut self) -> Result<(), Error> { Ok(()) }
    }
}
```

The macro injects a `pub host: Host` field on the storage struct and a `fn host(&self) -> &Host` accessor. `Host` is a cfg-gated wrapper: zero-sized type over `PolkaVmHost` on riscv64; `Rc<dyn HostApi>` on host-target builds so it can be cheaply cloned into helpers like `Lazy`/`Mapping`, and tests can construct the contract with a `MockHost`. On host targets the macro also emits a `Foo::with_host(backend: impl HostApi)` test constructor that wires up the storage fields against the backend without running the `#[constructor]` (seed state on the backend directly).

**DSL API** (explicit, manual dispatch):
```rust
let host = Host::new();
ContractBuilder::new()
    .method(BALANCE_OF_SELECTOR, balance_of_handler)
    .method(TRANSFER_SELECTOR, transfer_handler)
    .dispatch_impl::<256>(&host)
```

DSL handlers take a concrete `&Host` (same type the macro path injects on the storage struct). For typed cross-contract calls, handlers wrap a cloned host in `Context::new(host.clone())` — `Context` impls `ContractContext` so it can be passed to `.call(&mut cx)` / `.delegate_call(&mut cx)`. `Host::clone()` is `Copy` on riscv64 (ZST) and a single `Rc::clone` on host targets. Because the wrapper carries only the host handle (no storage state), the borrow checker cannot enforce view-vs-mutating in DSL; use the `#[contract]` macro path if you need that static guarantee. The same `Context` type is used in unit tests, where it owns a `Host` backed by a `MockHost`.

### Macro-Generated Code

The `#[contract]` macro generates two PolkaVM entry points:

- **`deploy()`** — calls the `#[constructor]` function

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paritytech/cargo-pvm-contract](https://github.com/paritytech/cargo-pvm-contract) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
