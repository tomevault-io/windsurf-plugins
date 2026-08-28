---
trigger: always_on
description: > Read this before making any changes. It captures non-obvious conventions,
---

# AGENTS.md — Notes for AI Agents Working on This Repo

> Read this before making any changes. It captures non-obvious conventions,
> gotchas, and design decisions that are easy to get wrong.

## Project at a Glance

`codexferry` is a local proxy daemon that lets Codex CLI (≥0.128) use
Chat-Completions-only upstreams (DeepSeek, Kimi, GLM, SiliconFlow, …) through a
single Responses-API endpoint. It routes by `provider/alias` model names,
converts between Responses ↔ Chat Completions protocols, and maintains
in-memory session state for cross-provider multi-turn conversations.

**Migration spec:** `docs/superpowers/specs/2026-08-22-codexferry-migration-design.md`
— this repo was migrated from `codex-router-rs` and renamed on 2026-08-22.

**On `spec §N` references in source comments:** ~114 comments across the codebase
cite sections of the *original* design doc (`spec §1` … `spec §14`), which was
not carried over in the migration. Those section numbers resolve against
`docs/superpowers/specs/2026-08-13-codex-router-design.md` in the retained
`codex-router-rs` repo. They are kept because they mark genuine provenance —
treat them as pointers to history, not to a file in this repo.

## Critical Conventions

### 1. Route key format is `provider/alias` — first `/` splits

Route keys like `deepseek/deepseek-v4-flash` are split on the **first** `/`
only. The prefix must match a `[providers.X]` key. Aliases may contain `/`
(e.g. `openai/o3-mini/high`). Use `split_once('/')`, never `split('/')`.

### 2. SSE parsing is hand-written — do not add dependencies

The SSE parser in `src/upstream.rs` (`parse_sse_stream`) is intentionally
dependency-free. It handles:
- `data:` lines (with or without space after colon)
- Multi-line `data:` within one event (joined with `\n`)
- Comment lines (`:` prefix, used as keepalive)
- `event:` / `id:` / `retry:` lines (ignored)
- `data: [DONE]` sentinel
- `\n\n` and `\r\n\r\n` event delimiters
- UTF-8 characters split across chunk boundaries (buffered at byte level)

If you touch the parser, run `cargo test upstream` — there are extensive
fixture-based tests including CRLF, split UTF-8, and trailing-event flushing.

### 3. Tracing format strings use `{field}` placeholders

Tracing macros auto-capture local variables via `{var_name}` in the message
string. For example `"session hit {id}"` captures the local `id` variable using
its `Value` impl. Do **not** confuse this with `format!`-style `{}` — positional
placeholders are not supported. For Display instead of Debug, use explicit field
assignment: `tracing::info!(id = %id, "session hit")`.

### 4. Responses-format passthrough keys sessions by upstream ID

For `format = "responses"` providers, the proxy relays SSE through — byte-for-byte
verbatim when the healing quirks (`dsml_heal`/`think_tags`) are off, and
event-granular healed (rewritten/injected) when they fire — and keys the
session store by the **upstream's** response ID (extracted from the
`response.completed` event), not a proxy-generated one. This is a deliberate
deviation from spec §8.4 ("代理生成自己的 response ID") because passthrough
means the client only ever sees upstream IDs (healing rewrites events in place
and never re-keys). A proxy-generated fallback key would be unreachable on the
next turn.

### 5. `previous_response_id` is consumed, never forwarded

The proxy strips `previous_response_id` from every request before forwarding to
any upstream. It uses the ID to look up stored conversation context from
`SessionStore`, merges it with new input, and converts the combined history. On
cache miss, it logs at `debug` level (in `SessionStore::get`) and degrades
gracefully (new input only, no crash).

### 6. Session storage is full-context snapshots (O(n²))

Each `response_id` stores the **complete** conversation context (all prior items
+ new input + new output), not incremental deltas. This is intentional for MVP
simplicity. Memory is bounded by TTL + LRU + `max_memory_mb`. See spec §8.2 and
§19 (risk: O(n²) storage growth). Requests carrying `store: false` skip
persistence entirely (the `store_enabled()` helper in proxy/capture.rs; only a literal
boolean `false` disables — absent or `true` stores as before): Codex sends
`store: false` and replays its full transcript inline every turn, so the
snapshot would never be read back.

### 7. Config hot-reload uses a channel + async applier (no lost updates)

The `notify` watcher callback runs on a synchronous thread. It uses
`tokio::sync::mpsc::unbounded_channel` to send the parsed config to an
applier task on the tokio runtime, which awaits the async `RwLock` write.
If the lock is busy (during an active request), the update is **queued** in
the channel and applied as soon as read guards release. Do not call a
blocking `write()` from the notify callback itself - it would stall the
notify thread behind every in-flight request. The channel indirection keeps
the notify thread free while guaranteeing delivery.

### 8. Tool calls are accumulated during streaming, emitted at stream end

Streaming tool calls arrive as deltas keyed by `index`. The `StreamConverter`
accumulates them silently in a `BTreeMap<usize, (id, name, arguments)>` during
the stream (no events emitted). Function-name deltas are handled

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nearbyfly/codexferry](https://github.com/nearbyfly/codexferry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
