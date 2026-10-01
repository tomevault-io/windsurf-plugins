---
trigger: always_on
description: `kiro-lb` is a Rust gateway (axum + tokio) that exposes Kiro (Amazon Q
---

# PROJECT KNOWLEDGE BASE

## OVERVIEW

`kiro-lb` is a Rust gateway (axum + tokio) that exposes Kiro (Amazon Q
Developer / CodeWhisperer) through OpenAI- and Anthropic-compatible APIs, load
balances across a pool of Kiro accounts, and ships a React operations dashboard
embedded in the binary. **AGPL-3.0**: a Rust rewrite of the Python port of
`jwadow/kiro-gateway`; see `LICENSE` and `NOTICE.md` (do not relicense).

The Python implementation was removed. Its SQLite schema, `/v1` wire format,
`/api/dashboard` JSON, session cookie and scrypt key format are kept
byte-compatible, so an existing `data/dashboard.sqlite3` opens with no migration.

## STRUCTURE

```
kiro-lb/
├── Cargo.toml               # crate kiro-lb, binary kirolb
├── .cargo/config.toml       # static CRT on Windows (no VC++ redistributable)
├── src/                     # gateway: 40 modules, ~15k lines
│   ├── main.rs              # CLI, router, startup, background tasks, banner
│   ├── bootstrap.rs         # first run: writes .env.example + .env with keys
│   ├── upstream/            # HTTP client, retries, endpoint rotation, proxies
│   └── docs.rs              # /docs (Swagger UI) + /openapi.json
├── static/                  # BUILD OUTPUT of frontend/, embedded via include_dir
├── frontend/                # Bun + Vite + React 19 dashboard source
├── tests/golden.rs          # replays tools/golden/corpus recorded from Python
├── tools/golden/corpus/     # frozen Python oracle outputs (*.json.gz)
├── data/                    # dashboard.sqlite3 (gitignored)
├── deploy/                  # Grafana dashboard, Pushgateway units, blue/green
├── Dockerfile               # packages CI-built dist/kirolb-linux-* binaries
└── docker-compose*.yml      # default and homelab (edge HAProxy + 2 slots)
```

## WHERE TO LOOK

| Task | Location | Notes |
|---|---|---|
| Client endpoints | `src/routes_v1.rs` | messages, count_tokens, chat/completions, responses, models |
| Responses API (Codex CLI) | `src/convert_responses.rs`, `src/stream_responses.rs` | Facade over chat completions |
| Request -> Kiro payload | `src/convert_core.rs` | Both adapters delegate here |
| Kiro -> client stream | `src/stream_openai.rs`, `src/stream_anthropic.rs` | Shared events in `src/stream_core.rs` |
| AWS event-stream framing | `src/parser.rs` | Frame reassembly + bracket tool-call recovery |
| Account failover / routing | `src/pool.rs` | Circuit breaker, weighted/sticky/most_credits/session |
| Credentials + hosts | `src/auth.rs`, `src/config.rs` | Builder ID routes to a different host |
| Device login | `src/device_login.rs` | Social + Builder ID flows |
| SQLite persistence | `src/store.rs`, `src/dashboard_store.rs` | WAL, additive migrations |
| Dashboard API | `src/routes_dashboard.rs` | `/api/dashboard/*`, `/metrics`, handoff |
| Runtime settings | `src/settings.rs` | Tunables, agent mode, prompt filter |
| Prometheus exposition | `src/metrics.rs` | Model labels clamped |
| Token accounting | `src/usage_tracking.rs` | Per key, account, model; batched flush |
| Token counting | `src/tokenizer.rs` | Per-family encoding + CJK-only correction |
| Model names | `src/model_resolver.rs` | Never rejects; unknown names pass through |
| Payload guard | `src/payload_guard.rs` | cl100k tokens of the compact JSON |
| Claude Code prompt reduction | `src/prompt_filter.rs` | Condense + shorten tool descriptions |
| Debug capture / replay | `src/debug.rs` | `kirolb replay <capture>` |

## CONVENTIONS

- Reasoning is forwarded only from native upstream frames. Never synthesize it
  from response text or prompt tags.
- OpenAI reasoning is emitted as `reasoning`, not `reasoning_content`; requests
  accept both.
- Any new client-visible behavior lands on OpenAI **and** Anthropic, streaming
  and non-streaming.
- `/v1/responses` is a translation facade over chat completions, not a third
  pipeline: failover, payload building and token accounting are inherited.
- The Codex CLI declares tools in an `additional_tools` input item, and its
  shell is a `custom` freeform tool. It is bridged as a function with one string
  field and unwrapped back into `custom_tool_call`. Dropping it made the model
  invent command output.
- Each protocol's usage object carries only the fields that protocol defines.
- Token counting: `cl100k_base` for Claude and unknown names, `o200k_base` for
  GPT/o-series and deepseek/qwen/minimax/glm. The CJK correction is a property of
  the script and is 1.0 for Latin text.
- Per-key token rows are keyed by the normalized model name.
- Output speed follows the benchmark definition (Artificial Analysis, IETF
  TPOT): `(output_tokens - 1) / (t_last - t_first)` between the first and last
  output event. Time to first token is stored separately (`request_logs.ttft_ms`)
  and never mixed into speed. `generation_ms` is that decode window, and
  `timed_completion_tokens` adds `completion - 1` so the `/metrics` ratio
  matches. Non-streaming responses have no per-token timing and record no speed.
- Control and data planes stay separate: dashboard sessions cannot call `/v1`,
  `/v1` keys cannot call `/api/dashboard`. `/metrics` takes a `/v1` bearer key.
- The SQLite store never holds prompts, completions or raw client keys. New

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [minpeter/kiro-lb](https://github.com/minpeter/kiro-lb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
