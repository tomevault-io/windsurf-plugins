---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Juno is a sovereign Cosmos SDK / CometBFT chain with CosmWasm smart-contract support (via `wasmd`). The binary is `junod`. Go module path is `github.com/CosmosContracts/juno/v30` — the `/v30` suffix tracks the current chain upgrade name and is bumped each consensus-breaking release (see `RELEASES.md`).

## Toolchain

Go 1.25.2 is pinned in `.mise.toml` (also: `buf`, `yq`). Run `mise install` once to provision them. Dev tools (`golangci-lint`, `gofumpt`, `buf`, protoc plugins) are declared as Go tool dependencies in `go.mod` and invoked via `go tool …` — do not install them separately.

## Common commands

Run from the `juno/` submodule root:

```bash
make build              # → ./bin/junod
make install            # → $GOPATH/bin/junod
make lint               # go tool golangci-lint run --config ./.golangci.yml
make format             # go tool gofumpt -l -w .
make init               # sh scripts/init.sh — wipes ~/.juno and bootstraps a single-validator chain
go test ./...           # unit tests
go test ./x/clock/keeper -run TestEndBlocker -v   # single test pattern
```

Local devnet (Docker, single node, exposes 1317/26656/26657/9090):

```bash
STAKE_TOKEN=ujunox UNSAFE_CORS=true TIMEOUT_COMMIT=1s docker-compose up
```

`docker-compose.yml` runs `scripts/init.sh && scripts/start.sh` inside the container; `scripts/init.sh` is also the source of truth for genesis bootstrap (default denom `ujunox`, chain-id `testchain-1`, keyring `test`).

## Interchaintest (e2e)

End-to-end tests live in `interchaintest/tests/<suite>/` and use Strangelove's interchaintest framework — they spawn real chains in Docker and are slow. The repo is a separate Go module (`interchaintest/go.mod`). Each suite has a `make ictest-<name>` target that runs `go test -race -v -run <Suite> .` after `go clean -testcache`:

`ictest-basic`, `ictest-cw`, `ictest-node`, `ictest-feemarket`, `ictest-fees`, `ictest-upgrade`, `ictest-ibc`, `ictest-ibc-hooks`, `ictest-pfm`, `ictest-tokenfactory`, `ictest-drip`, `ictest-burn`, `ictest-fixes`.

Most suites need a local image — build it first with `make local-image` (uses `docker buildx`).

## Protobuf

Proto generation runs inside a dedicated Docker image — there is no host-toolchain path:

```bash
make proto-image        # build juno-protobuilder:latest (one-time / when proto/Dockerfile changes)
make proto-gen          # proto-gogo + proto-pulsar + proto-openapi
make proto-check        # proto-format + proto-lint
make proto-breaking     # diffs ./proto against origin/main
```

`.proto` sources live in `proto/juno/` and `proto/osmosis/` (tokenfactory inherits Osmosis package paths). OpenAPI specs are wired into the chain's REST layer via `app/endpoints/openapi*.go`.

## Architecture

### App wiring (`app/`)

`app/app.go` defines `App` and assembles everything. The wiring is split into a few intentional pieces — change the matching one rather than `app.go`:

- `app/keepers/keepers.go` — `AppKeepers` struct and `NewAppKeepers(...)`. All module keepers (SDK, IBC, wasm, custom Juno modules) are constructed here, store keys live in `keys.go`, and `wasm_config.go` / `acceptedQueries.go` configure the wasm VM and stargate query allow-list.
- `app/modules.go` — module manager registration, ordering of `BeginBlocker` / `EndBlocker` / `InitGenesis`, and ModuleAccount permissions. Adding a module means touching this file.
- `app/ante/ante.go` — custom `AnteHandler` chaining SDK ante decorators with `wasm`, IBC, and the Juno fee stack (`feemarket`, `feepay`, `feeshare`). `app/ante/decorators/` and `app/ante/msg_filter.go` hold the Juno-specific decorators. `app/post.go` has post-handlers.
- `app/upgrades/` — one subpackage per named upgrade (currently `v30/`), each exporting an `Upgrade` value combining `UpgradeName`, a `CreateUpgradeHandler` constructor, and `StoreUpgrades` (added/deleted/renamed module stores). `app.Upgrades` lists them and `app.go` registers handlers via the upgrade keeper. `v30` deletes legacy `globalfee`, `crisis`, `params`, `nft` stores and adds `feemarket`.
- `app/endpoints/` — REST/OpenAPI/Scalar docs and the websocket endpoint (`endpoints/websocket/`) bolted onto the cosmos-sdk API server.
- `cmd/junod/` — CLI entry (root command, server commands, genesis subcommands like `add-ica-config`).

### Custom `x/` modules

Each follows the standard SDK module layout (`keeper/`, `module/`, `types/`, often `spec/` and a top-level `README.md`). Module summary:

- `x/clock` — registers smart contracts to be `sudo`-executed each `EndBlock` (no external bots).
- `x/cw-hooks` — fires staking/validator lifecycle events into registered contracts.
- `x/feemarket` — AIMD EIP-1559 dynamic base-fee module (originally Skip's). Wired into the ante chain.
- `x/feepay` — gasless UX: contract devs fund a balance that pays user tx fees on their contract.
- `x/feeshare` — directs a share of contract execution fees to the contract's registered withdraw address (Juno Prop 51; based on Evmos x/revenue). Has its own ante decorator at `x/feeshare/ante/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CosmosContracts/juno](https://github.com/CosmosContracts/juno) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
