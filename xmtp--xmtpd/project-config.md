---
trigger: always_on
description: Database layer using **sqlc** for query generation, **pgx v5** driver, and **golang-migrate** for schema migrations. Supports read/write replica routing via `Handler`.
---

# CLAUDE.md — pkg/db

## Overview

Database layer using **sqlc** for query generation, **pgx v5** driver, and **golang-migrate** for schema migrations. Supports read/write replica routing via `Handler`.

## File Layout

```
pkg/db/
  sqlc/           # source .sql query files (input to sqlc)
  queries/        # generated Go code (output of sqlc) — do not edit
  types/          # custom Go types (e.g. GatewayEnvelopeBatch)
  db.go           # Handler (read/write routing)
  pgx.go          # connection management, NewNamespacedDB
  tx.go           # transaction helpers
pkg/db/migrations/   # SQL migration files (up/down)
sqlc.yaml         # sqlc config (project root)
```

## Schema Summary

| Table                         | Purpose                                                | Key Columns                                                                                                   |
| ----------------------------- | ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------- |
| `gateway_envelopes_meta`      | Envelope metadata (hot path, partitioned)              | `originator_node_id`, `originator_sequence_id`, `topic`, `payer_id`, `expiry`, `gateway_time`                 |
| `gateway_envelopes_blob`      | Envelope payloads (cold path, partitioned)             | `originator_node_id`, `originator_sequence_id`, `originator_envelope`                                         |
| `gateway_envelopes_view`      | View joining meta + blobs                              | —                                                                                                             |
| `gateway_envelopes_latest`    | Latest sequence ID per originator (trigger-maintained) | `originator_node_id`, `originator_sequence_id`, `gateway_time`                                                |
| `staged_originator_envelopes` | Publish queue before ordering                          | `id` (serial), `topic`, `payer_envelope`                                                                      |
| `node_info`                   | Local node identity (singleton)                        | `node_id`, `public_key`                                                                                       |
| `address_log`                 | Address → inbox_id mapping                             | `address`, `inbox_id`, `association_sequence_id`, `revocation_sequence_id`                                    |
| `payers`                      | Payer addresses                                        | `id`, `address`                                                                                               |
| `unsettled_usage`             | Per-payer per-originator usage tracking                | `payer_id`, `originator_id`, `minutes_since_epoch`, `spend_picodollars`                                       |
| `payer_reports`               | Cross-node payer settlement reports                    | `id`, `originator_node_id`, `start_sequence_id`, `end_sequence_id`, `submission_status`, `attestation_status` |
| `payer_report_attestations`   | Attestation signatures on reports                      | `payer_report_id`, `node_id`, `signature`                                                                     |
| `payer_ledger_events`         | Deposit/withdrawal/settlement events                   | `event_id`, `payer_id`, `amount_picodollars`, `event_type`                                                    |
| `blockchain_messages`         | Links envelopes to blockchain blocks                   | `block_number`, `block_hash`, `originator_node_id`, `originator_sequence_id`, `is_canonical`                  |
| `latest_block`                | Last indexed block per contract                        | `contract_address`, `block_number`                                                                            |
| `originator_congestion`       | Per-originator message rate tracking                   | `originator_id`, `minutes_since_epoch`, `num_messages`                                                        |
| `nonce_table`                 | Transaction nonce management                           | `nonce`                                                                                                       |
| `migration_tracker`           | Data migration progress tracking                       | `source_table`, `last_migrated_id`                                                                            |
| `migration_dead_letter_box`   | Failed migration records for retry                     | `source_table`, `sequence_id`, `payload`, `reason`, `retryable`                                               |

## Partitioning

`gateway_envelopes_meta` and `gateway_envelopes_blob` use two-level partitioning:

1. **Level 1 — LIST** by `originator_node_id`: one child table per originator (e.g. `gateway_envelopes_meta_o100`)
2. **Level 2 — RANGE** by `originator_sequence_id`: 1M-row bands (e.g. `gateway_envelopes_meta_o100_s0_1000000`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xmtp/xmtpd](https://github.com/xmtp/xmtpd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
