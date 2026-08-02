---
trigger: always_on
description: This is the official Sentry SDK for Elixir. It captures errors, monitors cron jobs, and supports distributed tracing via OpenTelemetry integration. The SDK is built as an OTP application with pluggable HTTP clients, logger integration, and framework support for Plug and Phoenix.
---

# Sentry Elixir SDK - Agent Guide

## Overview

This is the official Sentry SDK for Elixir. It captures errors, monitors cron jobs, and supports distributed tracing via OpenTelemetry integration. The SDK is built as an OTP application with pluggable HTTP clients, logger integration, and framework support for Plug and Phoenix.

## Project Structure

| Path | Description |
|------|-------------|
| `lib/sentry.ex` | Main public API (`capture_exception`, `capture_message`) |
| `lib/sentry/application.ex` | OTP Application supervisor |
| `lib/sentry/client.ex` | Event creation, sampling, and callbacks |
| `lib/sentry/config.ex` | Configuration validation and persistence |
| `lib/sentry/transport.ex` | HTTP transport abstraction |
| `lib/sentry/transport/` | Sender pool, rate limiter |
| `lib/sentry/telemetry/` | Ring buffers, scheduler, queue worker |
| `lib/sentry/telemetry_processor.ex` | Supervisor for buffered event processing |
| `lib/sentry/opentelemetry/` | OpenTelemetry span processor, sampler, propagator |
| `lib/sentry/integrations/` | Oban, Quantum, Telemetry integrations |
| `lib/sentry/logger_handler.ex` | Erlang logger handler |
| `lib/sentry/plug_capture.ex` | Plug exception capture |
| `lib/sentry/live_view_hook.ex` | Phoenix LiveView hook |
| `lib/mix/tasks/` | Mix tasks (install, test event, source packaging) |
| `test/` | Unit and integration tests |
| `test/support/` | Test helpers and shared utilities |
| `test_integrations/` | Multi-project integration tests (umbrella, phoenix_app, legacy_otel, tracing) |
| `config/` | Application configuration |
| `pages/` | Documentation pages |

## Environment & Setup

- **Elixir:** ~> 1.13
- **Install dependencies:** `mix deps.get`
- **Compile:** `mix compile`

## Key Commands

| Command | Description |
|---------|-------------|
| `mix deps.get` | Install dependencies |
| `mix compile` | Compile the project |
| `mix test` | Run the test suite |
| `mix format` | Format code |
| `mix format --check-formatted` | Check formatting (CI) |
| `mix dialyzer` | Run Dialyzer type checking |
| `mix test.integrations` | Run integration tests in `test_integrations/` |

## Architecture

### OTP Application Tree (`Sentry.Application`)

The SDK starts as an OTP application with a `:one_for_one` supervisor. Key children:

1. **Registry** — `Sentry.Transport.SenderRegistry` for sender worker lookup
2. **Sentry.Sources** — Source code context for error reports
3. **Sentry.Dedupe** — Event deduplication
4. **Sentry.ClientReport.Sender** — Reports discarded events to Sentry
5. **HTTP Client** — Pluggable (Finch or Hackney), started if it has `child_spec/0`
6. **Sentry.OpenTelemetry.SpanStorage** — ETS-backed span storage (when tracing is enabled)
7. **Sentry.TelemetryProcessor** — Buffered event processing pipeline
8. **Sentry.Transport.RateLimiter** — Sentry rate limit handling (disabled in test)
9. **Sentry.Transport.SenderPool** — Pool of HTTP sender workers

After children start, integration handlers (Oban, Quantum, Telemetry) are attached and the logger handler is conditionally added.

### TelemetryProcessor Pipeline

The `TelemetryProcessor` is a supervisor managing the buffered event pipeline:

- **4 Ring Buffers** — One per data category: error, check_in, transaction, log
- **Scheduler** — Weighted round-robin scheduling (error=5, check_in=4, transaction=3, log=2)
- **QueueWorker** — Bounded FIFO queue (default capacity 1000) between scheduler and HTTP transport

Events flow: `Client → Buffer → Scheduler → QueueWorker → Transport`

### Send Modes

- **`:sync`** — Events sent directly via Transport (used in tests)
- **`:none`** — Events buffered through the TelemetryProcessor pipeline (production default)

> **Next major release:** The TelemetryProcessor will become the sole mechanism for buffering, scheduling, and prioritizing all event types (errors, transactions, and logs). The `:sync` send mode and the legacy `SenderPool` path will be removed, making the TelemetryProcessor pipeline the only and default way events are processed and delivered.

### Configuration

Configuration is validated at application start using NimbleOptions, then cached in `:persistent_term`. Runtime changes are possible via `Sentry.put_config/2`.

## Testing Conventions

### Framework & Organization

- **Framework:** ExUnit
- **File naming:** `*_test.exs` files mirror `lib/` structure under `test/`
- **Test helpers:** `test/support/case.ex` (`Sentry.Case`) and `test/support/test_helpers.ex`
- **Fixtures:** `test/fixtures/` for umbrella app test data

### Key Patterns

- **HTTP testing:** Use `Bypass` for HTTP-level tests with `send_result: :sync`. Use `setup_bypass/1` to open a Bypass instance and configure DSN, `setup_bypass_envelope_collector/1` to forward envelopes to the test process, and `collect_envelopes/3` + `extract_events/1` / `extract_transactions/1` / `extract_log_items/1` to retrieve and filter decoded envelope items.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getsentry/sentry-elixir](https://github.com/getsentry/sentry-elixir) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
