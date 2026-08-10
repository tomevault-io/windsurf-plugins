---
trigger: always_on
description: > Pure-Rust port of CerebroCortex. Same 67 MCP tools, same wire format, zero Python runtime.
---

# CerebroCortex-RS — Agent & Developer Guide

> Pure-Rust port of CerebroCortex. Same 67 MCP tools, same wire format, zero Python runtime.
> Drop-in for ApexOS `plugins.toml`. Pi 5 native. Single binary.

See also: [ARCHITECTURE.md](ARCHITECTURE.md) | [CONTRIBUTING.md](CONTRIBUTING.md)

---

## What this is

`cerebro-mcp` is a Cargo workspace of four crates:

```
crates/
  cerebro/        # library — all cognitive logic (types, models, activation, engines, storage)
  cerebro-mcp/    # MCP-over-stdio binary (67 tools) — the ApexOS drop-in
  cerebro-api/    # axum REST API + the Lucida observatory (ui-web/ embedded, served at /)
  cerebro-cli/    # clap CLI (mirrors Python cerebro CLI)
ui-slint/         # cerebro-ui — Lucida's native mirror (Slint; dashboard + Atlas + Thought)
```

Full design and build order: [ARCHITECTURE.md](ARCHITECTURE.md).
Reference implementation: `../CerebroCortex` (Python — **do NOT modify**).

---

## Locked decisions

- **Language**: Rust, workspace of 4 crates
- **Storage**: single SQLite file — sqlite-vec (ANN), FTS5 (keyword fallback), petgraph (in-memory graph rebuilt from SQLite on init)
- **Embeddings**: fastembed (ONNX Runtime, 384-dim, ~33MB model, no GPU required)
- **MCP**: hand-rolled newline-delimited JSON-RPC over stdio, protocol `"2024-11-05"` — no SDK dependency
- **LLM (dream engine)**: reqwest → Anthropic API directly, same client pattern as agentd
- **Python original**: stays untouched until this port passes the full test suite

---

## Pi 5 target

| Detail | Value |
|--------|-------|
| SSH | `ssh apexos@192.168.0.158` (LAN only, pw: `abnudc1337`) |
| OS | Debian trixie headless |
| Storage | NVMe `/dev/sda2`, 458 GB |
| RAM | 8 GB |
| Binary | `/usr/local/bin/cerebro-mcp` |
| Data dir | `/var/lib/cerebro/` (`CEREBRO_DATA_DIR=/var/lib/cerebro`) |
| Service | `/etc/systemd/system/cerebro.service` (from `deploy/cerebro.service`) |
| Env file | `/etc/cerebro/env` — plain `KEY=VALUE`, no `export`, chmod 600 root-owned |

**Always build on the Pi — never cross-compile.**
The Pi is Cortex-A76 (arm64). An x86 binary gives "Exec format error". Pi 5 builds in ~2 min.

---

## Deploy workflow (commit → push → Pi)

```bash
# 1. Dev machine — code passes tests, then:
cargo test
git add -p
git commit -m "short imperative description"
git push

# 2. On Pi
cd ~/CerebroCortex-RS
git pull
cargo build --release -p cerebro-mcp

# 3. Hot-swap the binary (running binary = "text file busy" — always stop first)
sudo systemctl stop cerebro-mcp
sudo cp target/release/cerebro-mcp /usr/local/bin/cerebro-mcp
sudo systemctl start cerebro-mcp

# 4. Verify
sudo journalctl -u cerebro-mcp -n 20 --no-pager
```

**One-liner for a code-only change:**
```bash
sudo systemctl stop cerebro-mcp && \
cargo build --release -p cerebro-mcp && \
sudo cp target/release/cerebro-mcp /usr/local/bin/cerebro-mcp && \
sudo systemctl start cerebro-mcp && \
sudo journalctl -u cerebro-mcp -n 10 --no-pager
```

---

## ApexOS integration (the drop-in)

When `cerebro-mcp` is ready, one line in `/etc/agentd/plugins.toml` on the Pi:
```toml
[[plugin]]
id      = "cerebro"
cmd     = "/usr/local/bin/cerebro-mcp"   # was: python -m cerebrocortex.mcp
restart = "always"
```
`sudo systemctl reload agentd` (or `hot_reload_subsystem plugins` via the agent).
agentd never knows. Same 67 tools. Same MCP contract.

---

## Environment variables

| Var | Default | Purpose |
|-----|---------|---------|
| `CEREBRO_DATA_DIR` | `~/.cerebro-cortex/` | SQLite DB + exports root |
| `CEREBRO_EMBED_MODEL` | `BAAI/bge-small-en-v1.5` | fastembed model ID |
| `CEREBRO_API_ADDR` | `127.0.0.1:8765` | cerebro-api bind — non-loopback REQUIRES a token |
| `CEREBRO_API_TOKEN` | — | cerebro-api bearer/`?token=` auth (falls back to `AGENTD_TOKEN`) |
| `ANTHROPIC_API_KEY` | — | Required for dream engine LLM phases |
| `SQLITE_VEC_PATH` | system default | Path to `sqlite-vec` `.so` (TBD on Pi — `apt-cache show sqlite-vec`) |
| `RUST_LOG` | `info` | tracing filter — logs go to stderr, stdout is MCP JSON-RPC |
| `CEREBRO_VISION_BACKEND` | `auto` | describe_image VLM: `auto`\|`ollama`\|`anthropic`\|`off` (+ `_URL`/`_MODEL` for Ollama) |
| `CEREBRO_VISION_EMBED` | follows embed model | CLIP visual recall on/off (`search_vision`) |
| `CEREBRO_RETAIN_VERSIONS` / `_DREAM_REPORTS` / `_AUDIT_ROWS` | 10/90/20000 | retention caps, dream pre-phase sweep; 0 = keep forever |

---

## Build order (current progress)

| Step | Module | Gate | Status |
|------|--------|------|--------|
| 1 | `types.rs` + `models/` | Serde round-trips | ✓ 6 type tests pass |
| 2 | `activation/` | Values match Python fixtures within 1e-4 | ✓ 41 fixture tests pass |
| 3 | `storage/sqlite.rs` | Schema init, CRUD, scope filtering | ✓ 9 storage tests pass |
| 4 | `storage/vector.rs` | sqlite-vec loads, cosine search, FTS5 fallback | ✓ 7 vector tests pass |
| 5 | `storage/graph.rs` | petgraph rebuild + neighbor traversal | ✓ 5 graph tests pass |
| 6 | `engines/` (thalamus→neocortex) | All 8 deterministic engines pass | ✓ 34 engine unit tests pass |
| 7 | `cortex.rs` | `remember()` + `recall()` end-to-end | ✓ 6 cortex pipeline tests pass |
| 8 | `cerebro-mcp/` (core tools) | MCP handshake + remember/recall vs agentd | ✓ 8 dispatch tests pass |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [buckster123/CerebroCortex-RS](https://github.com/buckster123/CerebroCortex-RS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
