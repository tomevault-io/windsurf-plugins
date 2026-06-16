---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is aimx

Self-hosted email for AI agents. One binary, one setup command. Built-in SMTP server handles inbound. Direct SMTP delivery for outbound. aimx handles everything: ingest to Markdown, DKIM signing, MCP server, hooks (`on_receive` / `after_send`). `aimx serve` is the SMTP daemon. All other commands are short-lived processes.

## Build and test commands

```bash
# Build
cargo build
cargo build --release

# Install locally
cargo build --release && sudo cp target/release/aimx /usr/local/bin/

# Tests (all unit + integration)
cargo test

# Single test
cargo test test_name
cargo test -- --exact module::tests::test_name

# Lint
cargo clippy -- -D warnings
cargo fmt -- --check

# Format
cargo fmt
```

### Verifier service (separate Rust crate)

```bash
cd services/verifier
cargo build
cargo test
cargo clippy -- -D warnings
cargo fmt -- --check
```

CI runs both crates independently (`.github/workflows/ci.yml`).

### Test environment escape hatches

A handful of CLI gating points refuse to run as non-root in production. The
test suite injects an opt-in to keep the post-gate code paths exercised under
a non-root `cargo test` runner. Set this only from the test harness:

- **`AIMX_TEST_SKIP_AUTHZ_CHECK=1`** — bypasses the `authorize()` call
  (gating `Action::HookCrud`) in `aimx hooks --cmd` raw-shell paths so
  the rest of the command (config writes, fallback hints, error
  formatting) stays reachable without `sudo`. Read by `src/hooks.rs`
  only — mailbox CRUD authz is enforced server-side over UDS and never
  consults this env var. Production callers must never set this — it
  neutralizes the authz predicate by design.

Other test-only env vars are documented next to their read sites:
`AIMX_SANDBOX_FORCE_FALLBACK` (force the non-systemd-run hook executor),
`AIMX_CONFIG_DIR` / `AIMX_DATA_DIR` (redirect config + storage paths),
`AIMX_TEST_MAIL_DROP` (use the file-drop transport instead of MX delivery),
`AIMX_INTEGRATION_SUDO=1` (opt the integration suite into the root-only
MAILBOX-CRUD branch when the test runner has sudo).

## Architecture

### Two independent Rust crates

1. **`aimx`** (root `Cargo.toml`): the main CLI binary. Edition 2024.
2. **`aimx-verifier`** (`services/verifier/`): hosted verification service (axum HTTP + SMTP listener). Deployed separately with Docker. Edition 2021.

These are NOT a Cargo workspace. They have independent `Cargo.toml` files and `target/` directories.

### Main binary: subcommand dispatch

`main.rs` parses CLI via clap and dispatches to module-level `run()` functions. Each `src/*.rs` module owns one subcommand:

- `setup.rs`: `aimx setup [domain]`. Interactive setup wizard. Prompts for domain when omitted, generates TLS cert + DKIM keys, creates the `aimx-catchall` system user (sole reserved-uid for catchall mailbox storage), chowns each configured mailbox's `inbox/<name>/` and `sent/<name>/` to `<owner>:<owner> 0700`, installs systemd/OpenRC service file for `aimx serve` (runs as root; subprocesses drop to mailbox owner via `setuid` at fire time), displays colorized [DNS]/[MCP]/[Deliverability]/[User] sections, DNS retry loop, re-entrant detection. Writes the datadir README on completion. Requires root.
- `ingest.rs`: `aimx ingest`. Reads raw `.eml` from stdin (called by `aimx serve` in-process, or via stdin for manual use), parses MIME via `mail-parser`, writes Markdown with TOML frontmatter (`+++` delimiters) using `InboundFrontmatter` (section-ordered: Identity → Parties → Content → Threading → Auth → Storage), routes to `inbox/<mailbox>/`, extracts attachments into bundle directories, fires `on_receive` hooks.
- `send.rs`: `aimx send`. Thin UDS client. Composes an unsigned RFC 5322 message and submits it to `aimx serve` over `/run/aimx/aimx.sock` as one `AIMX/1 SEND` request frame. The client does NOT read `config.toml` or the DKIM key; signing, mailbox resolution, and MX delivery all live in `aimx serve`. Refuses to run as root. Exit codes: `0` OK, `1` daemon ERR, `2` socket-missing / connect failure / root, `3` malformed response.
- `send_protocol.rs`: `AIMX/1` wire protocol codec (shared by `SEND`, `MARK-READ`, `MARK-UNREAD`, `MAILBOX-CREATE`, `MAILBOX-DELETE`, `HOOK-CREATE`, and `HOOK-DELETE` verbs). Length-prefixed, binary-safe framing (`AIMX/1 <VERB>\n`, per-verb headers, `Content-Length:` header, body). Parses requests into a tagged `Request` enum and writes `SendResponse` / `AckResponse` frames. Pure async I/O; no filesystem or network. The daemon parses `From:` from the submitted body itself to resolve the sender mailbox (no client-supplied `From-Mailbox:` header). `HOOK-DELETE` carries a `Hook-Name:` header; the daemon resolves it against the effective name (explicit or derived).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uzyn/aimx](https://github.com/uzyn/aimx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
