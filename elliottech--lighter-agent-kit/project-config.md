---
trigger: always_on
description: >-
---


# Lighter Agent Kit

Trade on Lighter — a ZK-rollup perpetual futures and spot exchange.

Scripts live in this skill's `scripts/` directory. Read commands use `query.py`, write commands use `trade.py`. Every command prints structured JSON to stdout. Errors are always in JSON as `{"error": "..."}`.

Commands use `<group> <action>` syntax mirroring the Lighter UI (e.g. `market stats`, `order limit`, `orders open`).

For response schemas and details, see the [references/](references/) folder.

## Install

Copy this folder to one of the standard Claude Code skill locations:

- **Personal (all projects):** `~/.claude/skills/lighter-agent-kit/`
- **Project-level (this repo only):** `.claude/skills/lighter-agent-kit/`

On first call, `lighter-sdk` and its transitive deps will be installed into `<skill>/.vendor/pyX.Y/`. Every version is pinned by `requirements.lock`. Supported targets are Apple Silicon Macs plus Linux x86_64/arm64. **Intel** Macs are not supported because lighter-sdk doesn't ship a darwin-x86_64 signer binary.

Requires network egress to `mainnet.zklighter.elliot.ai` / `testnet.zklighter.elliot.ai`, `pypi.org` / `files.pythonhosted.org`, and `github.com` (pip clones the pinned `lighter-sdk` build from GitHub). On Claude.ai's sandbox the default egress allowlist doesn't include `github.com` — you'll need to expand it before first run.

Read commands work with no credentials. Write commands and account-private reads will first check environment variables and then your personal credentials file — see [references/env-vars.md](references/env-vars.md).

## How to Handle User Requests

1. **Symbol convention: perp-bare, spot-pair.** Perp markets use bare tickers (`BTC`, `ETH`, `SOL`, `LIT`); spot markets use quote-qualified pairs (`ETH/USDC`, `LIT/USDC`, `LINK/USDC`). The presence of `/` is the single discriminator — no `--market_type` flag is needed on symbol-resolved commands. Numeric `market_index` is also accepted as an escape hatch.
2. **Symbol resolution is automatic.** Symbols resolve from the live order-books API and are cached on disk for 5 minutes per host, so `query.py`, `trade.py`, and `paper.py` share the same market map across calls. Only when the user says something you can't confidently map (e.g. "brent oil", "gold", "silver"), run `python3 scripts/query.py market list --search <substring> [--market_type perp|spot]` first to discover it. These usually appear under ticker-style symbols like `BRENTOIL`, `XAU`, and `XAG`.
3. **Side accepts both forms.** `--side buy|sell|long|short` — both are accepted on perp and spot. Normalized internally to canonical (long/short for perp, buy/sell for spot).
4. **Filter at the source on high-cardinality reads.** `market funding`, `market stats`, and `market info` accept `--symbol` / `--market_index` / `--exchange` — always pass them when you only need one row.
5. Run the matching script: `python3 scripts/query.py <group> <action> ARGS` for reads, `python3 scripts/trade.py <group> <action> ARGS` for writes. Paths are relative to this skill's directory.
6. Parse the JSON and present it clearly.
7. **Account-scoped commands and `LIGHTER_ACCOUNT_INDEX`.** Two cases:
   - **Public reads** (`account info`, `account apikeys`) accept `--account_index` and default to `LIGHTER_ACCOUNT_INDEX` when omitted.
   - **Authenticated reads** (`account limits`, `portfolio performance`, `orders open`, `orders history`) are **self-only** using `LIGHTER_ACCOUNT_INDEX` instead of `--account_index` because the auth token is bound to that account.

**Diagnostic (optional):** if a call fails with an `import lighter` style error, run `python3 scripts/bootstrap.py` to verify the SDK vendored correctly. It should return a response with `status: ok` field. You should never need to run this in a healthy session.

---

## Read Commands (`query.py`)

### Public reads — no credentials

| Command | Purpose |
|---|---|
| `system status` | zkLighter system status (network id, timestamp). |
| `market list [--market_type perp\|spot] [--search X]` | Compact `{symbol, market_index, market_type}` catalog. Use for symbol → index lookups. |
| `market stats [--symbol X]` | Market overview: prices, 24h volumes, daily trades. Pass `--symbol` to get one row. |
| `market info [--market_type perp\|spot] [--symbol X]` | Full market metadata (fees, decimals, min sizes). Always filter; unfiltered is ~150 rows. |
| `market book <symbol> [--limit 20]` | Top-of-book bids and asks. `<symbol>` is perp ticker (`BTC`), spot pair (`ETH/USDC`), or numeric market_index. `--limit` is per side. |
| `market trades <symbol> [--limit 20]` | Recent fills for one market. `--limit` max 100. |
| `market candles <symbol> --resolution 1h --count_back 24` | OHLCV candles. Resolutions: `1m`, `5m`, `15m`, `30m`, `1h`, `4h`, `1d`. Returns `o`/`h`/`l`/`c` (price), `v` (base volume), `V` (quote volume), `t` (bucket start, ms), `i` (last trade id). |
| `market funding [--symbol X] [--market_index N] [--exchange X]` | Funding rates across venues. ~300 rows unfiltered; always pass `--symbol` or `--exchange`. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elliottech/lighter-agent-kit](https://github.com/elliottech/lighter-agent-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
