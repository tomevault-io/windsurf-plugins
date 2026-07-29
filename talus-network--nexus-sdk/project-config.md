---
trigger: always_on
description: This file documents the conventions and workflow expected of any AI agent
---

# Working on `nexus-sdk` — Agent context

This file documents the conventions and workflow expected of any AI agent
contributing to `nexus-sdk`. Read it once before touching code; refer back to
it when a pattern is unclear.

## Workspace layout

```text
nexus-sdk/                  cargo workspace root
├── sdk/                    `nexus-sdk` crate — Rust SDK
│   └── src/
│       ├── nexus/          high-level action types (TapActions, ToolActions,
│       │                   WorkflowActions, SchedulerActions, GasActions,
│       │                   NetworkAuthActions) + NexusClient, Crawler,
│       │                   Signer, gas pool, EventPoller, errors
│       ├── transactions/   PTB builders: tap.rs, tool.rs, workflow.rs,
│       │                   scheduler.rs, gas.rs, network_auth.rs
│       ├── types/          typed on-chain object/event models
│       │                   (TapRegistry, TapExecutionPayment, Tool, ToolRef,
│       │                   DagExecution, NexusObjects, derive helpers, …)
│       │                   plus Move-JSON / BCS serde parsers
│       ├── idents/         Move package/module/function/struct identifiers
│       │                   (primitives, sui_framework, workflow, tap, move_std)
│       ├── events/         event parsing
│       ├── test_utils/     sui_mocks (gRPC mocks), nexus_mocks (client mocks),
│       │                   containers (sui/redis testcontainers), faucet
│       ├── onchain_schema_gen/  Move-introspection schema generation
│       ├── signed_http/    application-layer Ed25519 signatures (feature)
│       ├── walrus/         Walrus storage client
│       ├── tool_fqn.rs     ToolFqn + `fqn!` macro
│       └── lib.rs          top-level re-exports
├── cli/                    `nexus-cli` crate — `nexus` binary
│   └── src/
│       ├── main.rs         top-level Cli + Command dispatch
│       ├── prelude.rs      common imports (GasArgs, JSON_MODE, sui)
│       ├── display.rs      command_title!, notify_success!, item!, loading!,
│       │                   json_output(), JSON_MODE: AtomicBool
│       ├── sui.rs          get_nexus_client(), gRPC client helpers
│       ├── cli_conf.rs     CliConf (~/.nexus/conf.toml)
│       └── {tool,conf,dag,scheduler,gas,tap,completion}/mod.rs
│                           subcommand groups with their handlers
├── toolkit-rust/           Rust toolkit (`nexus-toolkit`) for tool authors
├── helpers/                workspace helper crates / just recipes
├── docs/                   gitbook-synced docs (cli.md is the CLI reference)
├── target/                 cargo build output
├── Cargo.toml              workspace manifest
├── rustfmt.toml            unstable nightly-only options used
├── .nightly-version        pinned nightly toolchain for fmt
├── rust-toolchain.toml     stable toolchain for everything else
├── STYLE_GUIDE.md          markdown style rules (markdownlint-enforced)
├── CONTRIBUTING.md         contributor guide (pre-commit, commits, PRs)
└── CHANGELOG.md            keep-a-changelog, per-crate sections
```

Sibling repos checked out next to this one (paths depend on local layout):

- `nexus-next/` — on-chain Move packages (`sui/primitives`, `sui/interface`,
  `sui/registry`, `sui/workflow`), example TAPs (`sui/examples/demo_tap`),
  and the off-chain leader (`be/leader/`). Its `sui/bin/publish.sh` and
  `sui/bin/test_demo.sh` are the canonical localnet bring-up and demo driver.
- `nexus-tools/`, `nexus-workbench/`, `nexus-api/` — sibling repos consumed
  by docker-compose workbenches.

## SDK conventions

- **High-level actions** live in `sdk/src/nexus/<area>.rs`. Each action takes
  `&self` on the `*Actions` struct (held by `NexusClient`), submits via the
  shared signer/gas/crawler, and returns a typed `*Result` struct. Free-
  function `fetch_*` helpers (e.g. `fetch_registry`) live in the same
  file when they're useful without a full client.
- **PTB builders** live in `sdk/src/transactions/<area>.rs` and take
  `&mut TransactionBuilder` plus `&NexusObjects`. They never read from the
  network; they only emit move calls/inputs. Pair an action with one PTB
  builder per logical transaction.
- **Errors** flow through `NexusError` (`Wallet`, `Configuration`,
  `TransactionBuilding`, `Rpc`, `Parsing`, `Timeout`, `Channel`, `Storage`).
  Pick the variant that matches the root cause; only `Configuration` is
  string-typed.
- **On-chain decoding**: prefer `crawler.get_object::<T>` (JSON) for SDK
  types that implement `Deserialize` against Sui's Move-JSON, and
  `crawler.get_object_contents_bcs::<T>` for objects whose layout is best
  decoded from raw BCS. Dynamic field readers
  (`get_dynamic_fields_bcs`, `get_dynamic_object_fields`) sit on top of both.
- **ID derivation** uses `derive_tool_id`, `derive_tool_gas_id`,
  `derive_walk_execution_event_task_id`, etc. Never reimplement the
  ascii-string / BCS-blake2b derivation in shell or Python.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Talus-Network/nexus-sdk](https://github.com/Talus-Network/nexus-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
