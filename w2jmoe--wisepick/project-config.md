---
trigger: always_on
description: > **Docs:** [Overview](./README.md) | [Integration & SDK](./README_API.md) | [Agent Protocol](./AGENTS.md)
---

# AGENTS.md — WisePick Agent Automation Spec

> **Docs:** [Overview](./README.md) | [Integration & SDK](./README_API.md) | [Agent Protocol](./AGENTS.md)

**Audience:** Coding agents, runtime configurators, CI generators (Cursor, Cline, Devin, OpenHands, etc.).

**How to implement (SDK, env, adapters, deploy):** [README_API.md](./README_API.md) only.

**Product overview:** [README.md](./README.md).

**Default hosted endpoint:** `https://api.wishweaver.top` (early operator validation). Deployment and self-host setup: [INSTALL_FOR_AGENTS.md](./INSTALL_FOR_AGENTS.md).

**Boundary:** WisePick routes and scores; your runtime discovers, maps, executes, enforces policy, holds secrets and session state. WisePick remains agnostic to whether tools are registered via static manifests or dynamic MCP hosts.

**Product (today):** WisePick Decision API (WPDA) — deterministic capability routing and execution feedback via SDK/HTTP ([README_API.md](./README_API.md)). **Vision (future):** Execution Experience Network and collective decision memory across runtimes ([README.md](./README.md#-vision--愿景)).

```json
{
  "wise_pick": ["route", "explain", "persist_decision_id", "learn_from_feedback"],
  "agent_runtime": ["map_ecu", "execute", "enforce_policy", "retry", "secrets"]
}
```

---

## WisePick v1 ECU Protocol

HTTP paths remain `v0` (`/v1/decide`, `/v1/feedback`). **v1** denotes the capability-centric ECU payload agents must parse and act on.

### Decide request (agent → WisePick)

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `task` | string | yes | Natural-language intent for this routing cycle |
| `context` | object | no | Opaque key/value; may include `trace_id`, `session_id` for observability correlation |
| `constraints` | object | no | Opaque limits (cost, timeout, region flags, etc.) |

### ECU response (WisePick → agent)

| Field | Type | Required | Semantics |
| --- | --- | --- | --- |
| `decision_id` | string | yes | Persist until `feedback` or turn abandon |
| `capability_id` | string | yes | **What** work to perform—stable semantic type (see below) |
| `provider` | string | yes | **Which** implementation satisfies that capability for this decision |
| `execution_type` | enum | yes | **How** the runtime should invoke locally (see below) |
| `callable` | boolean | yes | `false` → no assumed direct invoke; **runtime fallback** (empty `provider` / `capability_id`); replan/enrich, do not tool-spray |
| `confidence` | number | yes | Router score (match + stats + bootstrap); not calibrated probability |
| `reason` | string | yes | Human-readable routing explanation |
| `explain` | object | yes | Audit/scoring detail |
| `trace` | object | yes | Timing, candidates, optional `yantrik_cluster` |
| `tool_key` | string | legacy | Mirrors `provider`; ignore for new configs |

### `capability_id` (semantic capability type)

- Stable identifier for a **class of work** (e.g. `audio_transcription`, `translation`, `search_files`).
- Primary lookup key in the agent’s capability registry and in OpenAI `tool_choice.function.name` when hard-routing the first completion.
- Must align with registered tool/MCP `function.name` values the model can call (see auto-config `tool_capability_map`).

### `provider` (implementation selector)

- Names the **vendor/backend** chosen for this `capability_id` (e.g. `feishu_minutes`, `openai`, `deepl`).
- Used **with** `capability_id` to select credentials, base URL, MCP server, or skill variant—not as a substitute for `capability_id`.
- Legacy alias: `tool_key` === `provider`.

### `execution_type` (local transport hint)

| Value | Agent obligation |
| --- | --- |
| `api` | Invoke via HTTP/SDK client owned by the runtime |
| `mcp` | Invoke via MCP tool call on a server the runtime maintains |
| `function_call` | Invoke in-process registered handler |

WisePick does not perform the invocation; the value only selects which executor branch to use after `(capability_id, provider)` resolution.

### Feedback request (agent → WisePick)

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `decision_id` | string | yes | From the ECU that initiated the execution |
| `success` | boolean | yes | Drives `tool_stats.success_rate` |
| `latency_ms` | integer | yes | Wall-clock execution duration (ms), `>= 0` |
| `token_cost` | object | no | `{ "input": int, "output": int }` — token ROI; aggregated as `avg_token_cost` |
| `result_quality` | number | no | `0.0`–`1.0` — subjective or automated quality signal |
| `actual_tool_used` | string | no | Tool/MCP **actually executed**; when set, `tool_stats` ROI attributes here instead of recommended `tool_key` |
| `user_note` | string | no | Free-text only (errors, context); do not embed structured ROI here |

### Closed loop (state machine)

```text
decide → store(decision_id, ecu) → resolve(capability_id, provider, execution_type) → execute → feedback → (next decide)
```

**Prohibited:** full tool/MCP enumeration after ECU; skipping `feedback`; random alternate tools when `callable` is true without new `task`/`context`.

---

## Observability contract (`mcp.route_decision.v1`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [w2jmoe/WisePick](https://github.com/w2jmoe/WisePick) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
