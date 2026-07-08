---
trigger: always_on
description: Collects Level-2 order book data from Binance (Spot + USDM Futures), Hyperliquid, Bybit v5 (Spot + Linear Perp), and dYdX v4 via WebSocket and writes Parquet files.
---

# fathom — Multi-Exchange LOB Collector

Collects Level-2 order book data from Binance (Spot + USDM Futures), Hyperliquid, Bybit v5 (Spot + Linear Perp), and dYdX v4 via WebSocket and writes Parquet files.

**Behavior contracts live in `specs/`** (capture matrix, schemas, streams, planned feeds) — check them before changing collection behavior. Decisions: `docs/adr/`.

## Commands

```bash
cargo build                  # build
cargo test                   # unit + integration + e2e (no smoke)
cargo clippy -- -D warnings  # lint
cargo fmt --check            # format check

make run                     # cargo run (uses config.toml)
make smoke                   # smoke tests against live Binance (manual only)

# Exchange-specific smoke tests (live network, manual only):
cargo test --test smoke_hl_test -- --include-ignored --test-threads 1 --nocapture
cargo test --test smoke_dydx_test -- --include-ignored --test-threads 1 --nocapture
```

## Architecture

```
Binance (spot/perp):
  WebSocket combined stream ({sym}@depth@100ms + {sym}@aggTrade;
                             perp adds {sym}@markPrice@1s + {sym}@forceOrder)
    → gap detection (per-symbol sequence validation)
    → apply diff to BTreeMap L2 book (src/orderbook/mod.rs)
    → OFI / churn / microprice accumulation (src/accumulator.rs)
    → aggTrade → trade tape + 1s buy/sell volume accumulation
    → markPrice/forceOrder → deriv feeds (perp only; no depth-liveness impact)
  Perp REST: openInterest polled per symbol every 60s → deriv feeds

Hyperliquid:
  WebSocket (single endpoint, subscribe after connect)
    → full L2 snapshots (~500ms) + trades + activeAssetCtx
    → snapshot-to-snapshot OFI / churn (src/connection/hyperliquid.rs)
    → accumulation via WindowAccumulator::on_diff_from_levels
    → activeAssetCtx → MarkFunding + OpenInterest per message

dYdX v4:
  WebSocket Indexer API (subscribe after connect)
    → initial snapshot + batched diffs (~250ms) + trades
    → local BTreeMap book (DydxBook in src/connection/dydx.rs)
    → accumulation via WindowAccumulator::on_diff_from_levels

Bybit v5 (spot/linear perp):
  WebSocket, subscribe after connect ({"op":"subscribe"}, spot batched <=10 args/request,
                                       linear one message: orderbook.1000 + publicTrade;
                                       linear adds tickers + allLiquidation)
    → WS-native snapshot (type:"snapshot", also server-initiated resync) + sequenced deltas
      (type:"delta", u-gap-checked — src/connection/bybit.rs::check_orderbook_gap)
    → apply diff to shared BTreeMap L2 book (src/orderbook/mod.rs, same as Binance/HL)
    → OFI / churn / microprice accumulation (src/accumulator.rs)
    → publicTrade → trade tape + 1s buy/sell volume accumulation (up to 1024 trades/message)
    → tickers → snapshot+delta ticker-state merge (src/connection/bybit_ticker.rs)
      → MarkFunding + OpenInterest on relevant field change (linear only; no REST OI poll)
    → allLiquidation → Liquidation rows (linear only)
  Client-detected gap or rejected subscribe-ack → reconnect + resubscribe + full per-symbol
    state reset (books + ticker-merge state), no REST re-snapshot call

All paths → four parallel writers + optional NATS sink:
  raw diff  → {data_dir}/raw/{exchange}/{symbol}/{date}/depth_HHMM_HHMM.parquet
  1s snap   → {data_dir}/1s/{exchange}/{symbol}/{date}/snap_HHMM_HHMM.parquet  (64 columns, 1 row/sec, hourly-rotated)
  trades    → {data_dir}/trades/{exchange}/{symbol}/{date}/trades_HHMM_HHMM.parquet  (Binance aggTrade + HL trades)
  deriv     → {data_dir}/deriv/{exchange}/{symbol}/{date}/{funding|oi|liq}_HHMM_HHMM.parquet  (hourly-rotated, perp venues)

Optional NATS streaming (src/nats_sink.rs):
  1s snap   → fathom.v1.{exchange}.{symbol}.snapshot  (FATHOM_SNAPSHOTS, file storage, critical)
  raw diff  → fathom.v1.{exchange}.{symbol}.depth     (FATHOM_DEPTH, file storage, 1h retention)
  trade     → fathom.v1.{exchange}.{symbol}.trade     (FATHOM_TRADES, file storage, 24h retention)
  deriv     → fathom.v1.{exchange}.{symbol}.{funding|oi|liq}  (FATHOM_DERIV, file storage, 24h retention;
              .funding carries MarkFunding — mark is folded in, no .mark subject)
```

**1s snapshot columns:** `ts_us`, `exchange`, `symbol`, `bid_px_0..9`, `ask_px_0..9`, `bid_sz_0..9`, `ask_sz_0..9`,
`mid_px`, `microprice`, `spread_bps`, `imbalance_l1`, `imbalance_l5`, `imbalance_l10`,
`bid_depth_l5`, `bid_depth_l10`, `ask_depth_l5`, `ask_depth_l10`,
`ofi_l1`, `churn_bid`, `churn_ask`, `intra_sigma`, `open_px`, `close_px`, `n_events`,
`volume_delta`, `buy_vol`, `sell_vol`, `trade_count`.

The trade columns (`volume_delta`, `buy_vol`, `sell_vol`, `trade_count`) are populated for all
exchanges: HL from the `trades` channel, Binance from `aggTrade` (attribution by taker side,
`is_buy = !is_buyer_maker`), Bybit from `publicTrade` (side given directly as `"Buy"`/`"Sell"`,
no maker/taker inversion needed, up to 1024 trades per WS message). See `specs/trades-feed.md`.

Derivatives feeds (`specs/derivatives-feeds.md`): Binance perp markPrice@1s + forceOrder +
OI REST poll; HL activeAssetCtx; Bybit linear `tickers` (snapshot+delta merge,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RigpaLabs/fathom](https://github.com/RigpaLabs/fathom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
