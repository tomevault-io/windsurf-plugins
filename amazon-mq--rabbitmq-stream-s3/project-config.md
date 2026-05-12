---
trigger: always_on
description: This file covers the `rabbitmq_stream_s3` plugin specifically. For general
---

# Instructions for AI Agents

This file covers the `rabbitmq_stream_s3` plugin specifically. For general
RabbitMQ conventions (comments, git, building, testing), see the parent
repository's `AGENTS.md` at the root of [`rabbitmq/rabbitmq-server`](https://github.com/rabbitmq/rabbitmq-server).

## Overview

`rabbitmq_stream_s3` provides tiered storage for RabbitMQ streams using Amazon
S3 as the remote tier. It uploads committed stream data as fragment objects and
maintains a manifest per stream. Consumers read from local data when available
and fall back to the remote tier for older data.

The plugin is hosted at [`amazon-mq/rabbitmq-stream-s3`](https://github.com/amazon-mq/rabbitmq-stream-s3).

## Documentation

- `docs/README.md` — design overview, glossary, data representation, operations
- `docs/manifest.md` — manifest structure, concurrency control, operations
- `docs/configuration.md` — all configuration keys with examples
- `docs/TODO.md` — known issues and planned work

Read the relevant doc before modifying any module.

## Repository Structure

### Source (`src/`)

| Module | Role |
|--------|------|
| `rabbitmq_stream_s3.erl` | Utility functions: key construction, index file helpers |
| `rabbitmq_stream_s3_api.erl` | Storage backend behaviour (callbacks for get, put, delete) |
| `rabbitmq_stream_s3_api_aws.erl` | AWS S3 backend implementation (production) |
| `rabbitmq_stream_s3_api_aws_pool.erl` | Connection pool for S3 HTTP connections; separate pools for reads and writes avoid reader starvation |
| `rabbitmq_stream_s3_api_fs.erl` | Filesystem backend implementation (tests) |
| `rabbitmq_stream_s3_app.erl` | OTP application module |
| `rabbitmq_stream_s3_array.erl` | Binary array operations (at, slice, partition_point, etc.) |
| `rabbitmq_stream_s3_db.erl` | Khepri interactions: stream ID storage, triggers, keep-while conditions |
| `rabbitmq_stream_s3_log_manifest.erl` | `osiris_log_manifest` behaviour: local tier interface, fragment recovery on boot |
| `rabbitmq_stream_s3_log_reader.erl` | `osiris_log_reader` behaviour: reads from local or remote tier |
| `rabbitmq_stream_s3_machine.erl` | Manifest state machine: fragment tracking, retention, groups |
| `rabbitmq_stream_s3_membership_reconciliation.erl` | `gen_server` implementing Continuous Membership Reconciliation (CMR) for streams |
| `rabbitmq_stream_s3_membership_reconciliation_events.erl` | `gen_event` handler that notifies the CMR server when membership should be evaluated |
| `rabbitmq_stream_s3_prometheus_collector.erl` | Prometheus metrics collector; implements the `prometheus_collector` behaviour |
| `rabbitmq_stream_s3_remote_reader.erl` | `gen_server` that pre-fetches and buffers data from the remote tier for a single consumer |
| `rabbitmq_stream_s3_remote_reader_sup.erl` | Supervisor for remote reader `gen_server` processes |
| `rabbitmq_stream_s3_request_metrics.erl` | Tracks S3 request duration as a histogram (set of per-bucket counters) |
| `rabbitmq_stream_s3_read_size_metrics.erl` | Tracks remote tier read size distribution as a histogram |
| `rabbitmq_stream_s3_histogram.erl` | Generic histogram used by `_api` (request duration) and `_remote_reader` (read size distribution) |
| `rabbitmq_stream_s3_server.erl` | `gen_server` coordinating uploads, deletions, manifest updates |
| `rabbitmq_stream_s3_sup.erl` | Top-level supervisor |

### Key relationships

- `rabbitmq_stream_s3_log_manifest` implements `osiris_log_manifest` — called by `osiris_writer` during segment rollover and chunk writes
- `rabbitmq_stream_s3_log_reader` implements `osiris_log_reader` — called by `rabbit_stream_reader` when a consumer subscribes
- `rabbitmq_stream_s3_server` is the central coordinator; `_log_manifest` notifies it of available fragments, it kicks off upload tasks
- `rabbitmq_stream_s3_machine` is a pure state machine (no side effects) that `_server` uses to track manifest state
- `rabbitmq_stream_s3_api` is the storage abstraction; `_api_aws` is the real backend, `_api_fs` is used in tests

### Include (`include/`)

`rabbitmq_stream_s3.hrl` — record definitions (`#fragment{}`, `#manifest{}`),
constants (`?INDEX_RECORD_SIZE_B`, `?CHUNK_HEADER_B`, `?MAX_FRAGMENT_SIZE_B`).

### Tests (`test/`)

| Suite | What it tests |
|-------|---------------|
| `config_schema_SUITE.erl` | Configuration schema snippets from `docs/configuration.md` |
| `membership_reconciliation_SUITE.erl` | Continuous Membership Reconciliation behaviour |
| `unit_SUITE.erl` | Property-based tests for `_array`, `find_fragment`, `find_index_position` |
| `s3_streams_SUITE.erl` | Integration tests: publish, read from remote tier by offset and timestamp |
| `rabbitmq_stream_s3_machine_SUITE.erl` | State machine transitions |
| `rabbitmq_stream_s3_api_aws_SUITE.erl` | AWS API client (requires credentials) |

## Building and Testing

```bash
# Build (including test dependencies)
gmake test-build

# Run all Common Test suites
gmake ct

# Run a specific suite
gmake ct-s3_streams

# Run a specific test case
gmake t=TEST_GROUP:TEST_CASE ct-s3_streams
```

Integration tests in `s3_streams_SUITE` use the filesystem backend (`_api_fs`)
and do not require AWS credentials or an S3 bucket.

## Key Concepts

### Fragment recovery on boot


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amazon-mq/rabbitmq-stream-s3](https://github.com/amazon-mq/rabbitmq-stream-s3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
