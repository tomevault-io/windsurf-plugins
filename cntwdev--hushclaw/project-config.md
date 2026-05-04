---
trigger: always_on
description: Guidance for Claude Code. Contains only what cannot be derived by reading the code —
---

# CLAUDE.md

Guidance for Claude Code. Contains only what cannot be derived by reading the code —
decisions, rationale, prohibitions, and open debt. Keep this file concise.

---

## Core Contract (non-negotiable)

1. **Zero mandatory dependencies** — `pip install hushclaw` on stock Python 3.11 must work. Optional features live behind `extras_require`. No non-stdlib import at module level outside `providers/anthropic_raw.py`, `providers/openai_raw.py`, and test files.
2. **`server_impl.py` is transport only** — HTTP/WebSocket dispatch. Zero business logic in `_handle_*` methods; each should delegate to a domain module.
3. **Tools are framework-agnostic** — no tool module may import from `hushclaw.*` except `tools.base`. All runtime dependencies arrive via parameter injection (`_memory_store`, `_config`, `_gateway`, `_session_id`, `_loop`, `_confirm_fn`, `_output_dir`, …).
4. **`event_stream()` is the primary API** — `run()` is a convenience wrapper.
5. **Compaction is never disabled** — fix bad summaries by improving compaction, not skipping it.

---

## Design Principles

### 1. Resilience First
Error taxonomy before recovery decisions (`hushclaw/core/errors.py`):
- `AUTH_FAILURE` → surface, stop | `RATE_LIMIT` → backoff + credential rotation
- `CONTEXT_TOO_LONG` → compact + retry | `TRANSIENT` → retry ×3 | `FATAL` → abort turn

Graceful degradation: vector search fails → BM25 only; browser unavailable → tools silently absent; trajectory write fails → log + continue. Never surface these to the user.

### 2. Safety by Design
- `_confirm_fn=None` blocks all dangerous tools by default (server path). REPL injects an interactive prompt.
- Subagent depth cap: parent (0) → child (1); children cannot re-delegate.
- SSRF protection in `fetch_url`: RFC 1918, loopback, link-local, cloud metadata IPs blocked before any socket opens (`tools/builtins/web_tools.py`).
- Skill sandboxing is **partial** — pip installs run unaudited in the same venv.

Priority when principles conflict: **Safety > Modularity > Token Economy > Observability**.

### 3. Token Economy
System prompt is a `(stable_prefix, dynamic_suffix)` tuple:
- `stable_prefix` — role + AGENTS.md + SOUL.md. KV-cache eligible (never changes mid-session).
- `dynamic_suffix` — today's date + USER.md + score-gated recalled memories (rebuilt every query).

Budgets are explicit (`context/policy.py`): `stable_budget`, `dynamic_budget`, `history_budget`, `memory_max_tokens`. No section silently crowds out another.

Memory recall: FTS shortcut (skip vector if BM25 ≥ 0.8) → score-gate → budget-cap → 30s session cache.

Skill loading is progressive: listing only → full SKILL.md → referenced files. Never load all skills at once.

### 4. Modularity via Composition
Four extension seams:
1. `LLMProvider.complete() → LLMResponse` — single-method contract, one file per provider.
2. `ContextEngine` ABC (`assemble`, `compact`, `after_turn`) — swap without touching AgentLoop.
3. `@tool` decorator — only registration mechanism.
4. Parameter-name injection — tools declare what they need by naming parameters.

**Parallel tool execution is implemented**: `ToolDefinition.parallel_safe=True` marks read-only tools. `event_stream()` in `loop.py` splits tool calls into dedup / parallel (`asyncio.gather`) / serial groups. Marked tools: `recall`, `search_notes`, `get_time`, `platform_info`, `read_file`, `list_dir`, `make_download_url`, `fetch_url`, `jina_read`.

### 5. Streaming and Observability
Event stream contract: `chunk`, `tool_call`, `tool_result`, `compaction`, `round_info`, `done`, `error`, `session`. Adding types is fine; removing/renaming is a breaking change.

Token accounting is first-class — persisted per turn and session to the `turns` table.

### 6. User Modeling & Learning
- `USER.md` — user profile (communication style, workflow, recurring goals). Injected into `dynamic_suffix`, distinct from `MEMORY.md` (world facts).
- Learning loop (`learning/controller.py`): captures tool traces per turn, runs `reflect_trace()`, persists to `reflections` + `skill_outcomes` tables. Quality score derived from trace: corrections → 0.0, errors → 0.6, clean → 1.0. Auto-patches single editable skills on strong signals.
- Memory creativity defaults enabled: `memory_decay_rate=0.002` (half-life ~350 days), `retrieval_temperature=0.1`.

---

## Web Access Stack

`fetch_url` priority: **curl_cffi** (Chrome TLS/h2 fingerprint, auto-installed) → **urllib fallback**.
Both paths: SSRF gate → request → CF challenge detection → decompress (gzip/deflate/brotli).
Proxy: reads `HTTP_PROXY` / `HTTPS_PROXY` / `ALL_PROXY` env vars; also accepts `proxies=` param.

Full web access priority for the LLM to use as a decision guide:
1. `fetch_url` — lightweight, TLS fingerprint, cookie jar, no JS rendering
2. `jina_read` — Jina proxy renders JS, returns clean markdown (200 req/day free)
3. `browser_navigate` + suite — full Playwright + playwright-stealth, handles Turnstile, login flows
4. `browser_connect_user_chrome` — CDP to user's existing Chrome (for already-logged-in sites)

---

## Skill Tiers

Four tiers, ascending priority (later overrides earlier):
| Tier | Directory | Who writes |
|------|-----------|------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CNTWDev/hushclaw](https://github.com/CNTWDev/hushclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
