---
trigger: always_on
description: Guidance for agents (and humans) working on this codebase. Read before adding features.
---

# lite-cli — architecture & conventions

Guidance for agents (and humans) working on this codebase. Read before adding features.

## What lite-cli is

A **thin, transparent logging proxy** that wraps Claude Code. It launches `claude` pointed at a
local proxy, forwards every request to the real upstream **unchanged**, and observes traffic
(tokens, model, latency, spend). It is an *observe/spend* tool — it never transforms requests or
responses.

## Prime directive: stay thin

Do the minimum needed to observe. Claude Code already sends its own credentials and request
bodies; the proxy forwards them verbatim. We only do two things claude can't do for us:

1. **Redirect** claude to the proxy (inject `ANTHROPIC_BASE_URL` via `--settings`).
2. **Know the real upstream** to forward to (resolve from env → `~/.claude/settings.json`).

If you're tempted to parse, rewrite, or re-inject something claude already provides, stop — that's
almost certainly the wrong layer. (We learned this the hard way: an auth-token re-injection was
added, then deleted once we confirmed claude sends `Authorization` itself.)

## Autorouter mode — the one sanctioned exception

`lite login` + `lite autorouter` write `~/.lite/settings.json` (gateway `api_base`/`api_key` + a
model per complexity tier: `simple`/`medium`/`complex`/`reasoning`). When **all six** fields are
present (`Settings::routing_enabled`), `lite claude` stops being verbatim and routes:

- Upstream is the gateway `api_base`.
- The proxy **parses the request body, rewrites `model`** to the tier model, and **injects the
  gateway api key** (`Authorization: Bearer …`, dropping claude's own creds). This is the *only*
  place we knowingly violate "stay thin" — it is gated entirely behind `state.routing.is_some()`;
  with no config the transparent path is byte-for-byte unchanged.
- **Classify-once-lock**: the first turn of a session (keyed by `x-claude-code-session-id`) decides
  the tier via `classifier.rs`, and that tier is held for the whole session to keep Anthropic
  prompt caching stable. The haiku small-fast slot always routes to `simple_model` and never sets
  the lock.
- `classifier.rs` is a faithful port of litellm's `router_strategy/complexity_router` (rule-based,
  local, zero API calls) plus three Claude Code signals: the `thinking` field (→ reasoning), tool
  count (gentle boost), and conversation context size.

Keep routing logic in `classifier.rs` (pure scoring) and `proxy.rs` (the gated rewrite). Don't
leak tier knowledge into pricing/log/presenters — they still just observe the *served* model.

## Layers — what goes where

Data flows: **claude → proxy → upstream**, with the proxy teeing usage into a log, which readers
render. Each module has one job:

| Module | Responsibility | Must NOT |
|---|---|---|
| `main.rs` | CLI parsing, process orchestration, env/settings resolution, launching `claude`, printing the session summary. | Contain HTTP, parsing, or pricing logic. |
| `proxy.rs` | Transparent HTTP forward. Extract raw signals from the wire (status, headers like `x-claude-code-session-id`, streaming-ness) and the token usage. In autorouter mode only (gated on `state.routing`), rewrite `model` + inject gateway auth. | Make business decisions beyond routing. No cost math, no aggregation. |
| `settings.rs` | Read/write `~/.lite/settings.json` (gateway creds + tier models, 0600). `routing_enabled` gate. | Network, pricing, HTTP. |
| `classifier.rs` | Pure complexity scoring (port of litellm `complexity_router` + CC signals) → `Tier`. | Touch network/disk/state. Stateless. |
| `login_cmd.rs` / `autorouter_cmd.rs` | Interactive setup: store creds; list gateway models and assign tier models. | Run during `lite claude`. Setup-only. |
| `usage.rs` | Pure parsing of token usage from responses (SSE `message_start`/`message_delta` + non-stream JSON). | Touch the network, disk, or pricing. |
| `pricing.rs` | The **pure cost model** — a faithful port of litellm's `generic_cost_per_token` (incl. tiered/threshold + service-tier + 5m/1h cache split via `cost_detailed`). Fetch/cache the price table; given tokens, return USD. | Know about requests, logs, or the dashboard. Stateless math + a cached table. |
| `transcripts.rs` | **Pure reader of Claude Code's own logs** (`~/.claude/projects/**/*.jsonl`). One `Turn` per billable assistant response, de-duped by `message.id`. The spend source of truth. | Price anything, render, or write. |
| `log.rs` | **Source of truth for the proxy's live log.** Computes `cost_usd` once (via `pricing`), writes one JSONL line per request, maintains the in-memory summary. | Render or present. |
| `logs_cmd.rs` | **Read-only presenter** of the proxy live log. Reads `~/.lite` JSONL, renders a table / tail. | Re-derive per-request cost. |
| `dashboard.rs` | **Read-only spend presenter.** Reads `transcripts`, prices each turn via `pricing`, aggregates by session / project / model. | Re-derive cost outside `pricing`; touch proxy state. |
| `dashboard.html` | Pure presentation. Visual transforms (cumulative sums, bar widths, formatting) live here. | Contain pricing or model knowledge. |

## Core rules

1. **Cost is computed once, at log time.** `log.rs` calls `pricing` and stamps `cost_usd` into the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LiteLLM-Labs/lite-cli](https://github.com/LiteLLM-Labs/lite-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
