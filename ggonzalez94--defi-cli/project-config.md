---
trigger: always_on
description: Short guide for agents working on `defi-cli`.
---

# AGENTS.md

Short guide for agents working on `defi-cli`.

## Project intent

`defi-cli` is an agent-first DeFi CLI for querying and acting on-chain. Core priorities are:

- stable JSON contract (envelope + fields + deterministic ordering)
- stable exit codes
- canonical IDs/amounts for automation (CAIP + base units)

## First 5 minutes

```bash
go build -o defi ./cmd/defi
go test ./...
go test -race ./...
go vet ./...

./defi providers list --results-only
./defi lend markets --provider aave --chain 1 --asset USDC --results-only
./defi lend positions --provider aave --chain 1 --address 0x000000000000000000000000000000000000dEaD --type all --limit 3 --results-only
./defi yield opportunities --chain 1 --asset USDC --providers aave,morpho --limit 5 --results-only
./defi yield positions --chain 1 --address 0x000000000000000000000000000000000000dEaD --providers aave,morpho --limit 5 --results-only
```

## Folder structure

```text
cmd/
  defi/main.go                    # CLI entrypoint

internal/
  app/runner.go                   # command wiring, provider routing, cache flow
  providers/                      # external adapters
    aave/ morpho/ moonwell/       # lending + yield (read + execution)
    defillama/                    # market/yield normalization + fallback + bridge analytics
    across/ lifi/                 # bridge quotes + lifi execution planning
    oneinch/ uniswap/ taikoswap/ tempo/ # swap quotes + execution planning providers
    types.go                      # provider interfaces
  execution/                      # action persistence + planner helpers + signer abstraction + tx execution
  registry/                       # canonical execution endpoints/contracts/ABI fragments + default chain RPC map
  config/                         # defaults + file/env/flags precedence
  cache/                          # sqlite cache + file lock
  id/                             # CAIP parsing + amount normalization
  model/                          # output envelope + domain models
  out/                            # json/plain rendering and field selection
  errors/                         # typed errors -> exit codes
  schema/                         # machine-readable command schema
  policy/                         # command allowlist
  httpx/                          # shared HTTP client/retry behavior

.github/workflows/ci.yml          # CI (test/vet/build)
.github/workflows/nightly-execution-smoke.yml # nightly execution planning drift checks
.github/workflows/release.yml     # tagged release pipeline (GoReleaser)
scripts/install.sh                # macOS/Linux installer from GitHub Releases
.goreleaser.yml                   # cross-platform release artifact config
assets/                            # static assets (logo, images)
docs/                             # Mintlify docs site
  docs.json                       # Mintlify docs site config
  *.mdx + concepts/ guides/ ...   # Mintlify docs content pages
README.md                         # user-facing usage + caveats
```

## Non-obvious but important

- Error output always returns a full envelope, even with `--results-only` or `--select`.
- Config precedence is `flags > env > config file > defaults`.
- `yield --providers` expects provider names (`aave,morpho,kamino,moonwell`), not protocol categories.
- Lending routes by `--provider` use direct protocol adapters (`aave`, `morpho`, `kamino`, `moonwell`).
- `lend positions` currently supports `--provider aave|morpho|moonwell`; `kamino` does not expose positions yet.
- `yield positions` currently supports `aave|morpho|moonwell`; `kamino` does not expose positions yet.
- `lend positions --type all` intentionally returns non-overlapping intents (`supply`, `borrow`, `collateral`) for automation-friendly filtering.
- Most commands do not require provider API keys.
- Key-gated routes: `swap quote --provider 1inch` (`DEFI_1INCH_API_KEY`), `swap quote --provider uniswap` (`DEFI_UNISWAP_API_KEY`), `chains assets`, and `bridge list` / `bridge details` via DefiLlama (`DEFI_DEFILLAMA_API_KEY`).
- Multi-provider command paths require explicit selector choice via `--provider`; no implicit defaults.
- Tempo quote/planning does not require an API key; execution uses native Tempo type 0x76 transactions via the TempoStepExecutor and currently settles Tempo DEX swaps back to the sender only.
- Tempo Stablecoin DEX swaps are currently USD TIP-20 only; the DEX auto-routes supported pairs through quote-token relationships, so non-USD assets should fail as `unsupported` rather than `unavailable`.
- TaikoSwap quote/planning does not require an API key; standard EVM/TaikoSwap execution supports `--wallet` (OWS, recommended) and `--from-address` (local signer) (`--private-key` override, `DEFI_PRIVATE_KEY{,_FILE}`, keystore envs, or auto-discovered `~/.config/defi/key.hex` / `$XDG_CONFIG_HOME/defi/key.hex`).
- `swap quote` (on-chain quote providers) and execution `plan` commands support optional `--rpc-url` overrides (`swap`, `bridge`, `approvals`, `transfer`, `lend`, `yield`, `rewards`); `submit`/`status` use stored action step RPC URLs.
- Execution `plan` and `submit` commands also support `--input-json` and `--input-file` (`-` reads stdin); explicit flags override structured input values.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ggonzalez94/defi-cli](https://github.com/ggonzalez94/defi-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
