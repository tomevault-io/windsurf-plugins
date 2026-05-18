---
trigger: always_on
description: Multi-chain liquidation bot for the Morpho Blue lending protocol. Monitors positions across all chains where Morpho is deployed and executes profitable liquidations via on-chain executor contracts.
---

# Morpho Blue Liquidation Bot

Multi-chain liquidation bot for the Morpho Blue lending protocol. Monitors positions across all chains where Morpho is deployed and executes profitable liquidations via on-chain executor contracts.

## Architecture

Workspace monorepo with six packages:

- **`apps/config`** — Chain configurations, venue/pricer/data-provider registrations, and all tunable parameters. Single source of truth for what the bot does and how.
- **`apps/client`** — Bot orchestration logic and on-chain execution. Contains no configuration or secrets — everything is injected from config.
- **`apps/data-providers`** — Data provider interface and implementations (MorphoApi, HyperIndex) for fetching market and position data.
- **`apps/hyperindex`** — Envio HyperIndex indexer package. Standalone service that indexes Morpho on-chain events. Used by the HyperIndex data provider.
- **`apps/liquidity-venues`** — Liquidity venue interface and implementations for converting collateral to loan tokens.
- **`apps/pricers`** — Pricer interface and implementations for pricing assets in USD.

### Key abstractions

- **`DataProvider`** (`apps/data-providers/src/dataProvider.ts`) — Interface for fetching market and position data. Multi-chain: a single instance is shared across all chains. Implements optional `init()`, `fetchMarkets`, and `fetchLiquidatablePositions`. Created in `script.ts` before bots launch.
- **`LiquidityVenue`** (`apps/liquidity-venues/src/liquidityVenue.ts`) — Interface for converting collateral to loan token. Venues are tried in order defined by config. Each venue implements `supportsRoute` and `convert`.
- **`Pricer`** (`apps/pricers/src/pricer.ts`) — Interface for pricing assets in USD. Used for profitability checks. Pricers are tried in order defined by config.
- **Factories** (`apps/data-providers/src/factory.ts`, `apps/liquidity-venues/src/factory.ts`, `apps/pricers/src/factory.ts`) — Map config string identifiers to class instances. The config package exports only string names; the implementation packages own the classes. The data provider factory (`createDataProviders`) takes chain IDs and returns a `Map<number, DataProvider>` with a shared instance.
- **`LiquidationBot`** (`apps/client/src/bot.ts`) — Core orchestrator. Fetches markets, finds liquidatable positions, encodes liquidation calldata, simulates, checks profitability, and executes.
- **`LiquidationEncoder`** (`apps/client/src/utils/LiquidationEncoder.ts`) — Builds batched calldata for the on-chain executor contract.

### Flow

1. Config defines which chains, data provider, vaults, venues, and pricers to use
2. `script.ts` reads all chain configs, groups chains by data provider, creates shared providers (awaiting `init()` for backfill), then launches one bot per chain
3. Each bot uses its data provider to fetch whitelisted markets and find liquidatable positions
4. For each position: try liquidity venues in order to convert collateral → loan token
5. Simulate the full liquidation, check profitability via pricers
6. Execute (optionally via Flashbots on mainnet)

## Non-Negotiables

- **Never commit secrets or private keys.** Secrets (RPC URLs, private keys, API keys) must come from environment variables. Never hardcode them anywhere.
- **All configuration lives in the config package.** The client, liquidity-venues, pricers, and data-providers packages must not define or hardcode any configuration within their own packages. All configuration (parameters, addresses, venue/pricer ordering, chain settings) lives in `apps/config`. These packages may access config values by importing directly from `@morpho-blue-liquidation-bot/config` — this is the intended pattern, not a violation. If you need a new parameter, add it to the config types in `apps/config`. These packages may also read secrets (e.g. RPC URLs, API keys) directly from environment variables.
- **Never push directly to `main`.** Always use feature branches and PRs.
- **Always run tests after code changes.** Run the relevant test suite before considering work complete.
- **Preserve venue/pricer ordering semantics.** The order of `liquidityVenues` and `pricers` arrays in config is significant — venues are tried sequentially and the first successful conversion wins. Pricers are tried in order and the first price found is used. `pricers` is optional — omitting it disables profitability checks for that chain.

## Code Standards

### TypeScript & viem

- Strict TypeScript. Use viem types (`Address`, `Hex`, `Chain`, `Transport`) throughout.
- Use `bigint` for all on-chain values. Never use `number` for token amounts, prices, or gas.
- Use `viem/actions` for chain interactions (`readContract`, `writeContract`, `simulateCalls`).
- Use `parseUnits`/`formatUnits` for decimal conversions — never manual `10 ** n`.

### BigInt precision

- Always be explicit about decimal precision when converting between units.
- Rounding direction matters: round in favor of the protocol (down for collateral, up for debt).
- `WAD = 10^18` is used as the fixed-point base. Use `wMulDown` from `utils/maths.ts`.

### Error handling


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [morpho-org/morpho-blue-liquidation-bot](https://github.com/morpho-org/morpho-blue-liquidation-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
