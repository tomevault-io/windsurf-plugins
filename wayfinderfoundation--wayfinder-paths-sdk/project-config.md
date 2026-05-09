---
trigger: always_on
description: This file provides guidance when working with code in this repository.
---

# AGENTS.md

This file provides guidance when working with code in this repository.

## First-Time Setup (Auto-detect)

**IMPORTANT: On every new conversation, check if setup is needed:**

1. **Detect Shells Instance first.** Probe `http://localhost:4096/global/health`. If it returns `{ "healthy": true, ... }`, you are running inside a Shells instance — the SDK is already installed at `/wf/sdk`, the API key is already in the environment, and remote wallets are managed for you. **Do NOT run `setup.py`, do NOT prompt for an API key, do NOT touch `config.json`** — proceed normally.

2. If `config.json` does NOT exist:
   - Run: `python3 scripts/setup.py`
   - After setup completes, ask the user: "Do you have a Wayfinder API key?"
     - If yes: Add it to `config.json` under `system.api_key`
     - If no: Direct them to **https://strategies.wayfinder.ai** to create an account and get one

3. If `config.json` exists but `system.api_key` is empty/missing AND `WAYFINDER_API_KEY` is not set:
   - Ask: "I see you haven't set up your API key yet. Do you have a Wayfinder API key?"
   - If yes: Help them add it to `config.json` under `system.api_key`
   - If no: Direct them to **https://strategies.wayfinder.ai** to get one

4. If everything is configured, proceed normally

## Wayfinder Shells Instance Environment Variables

When the SDK runs inside Wayfinder Shells, two env vars are injected at startup:

| Variable               | What it is                                                                             |
| ---------------------- | -------------------------------------------------------------------------------------- |
| `WAYFINDER_API_KEY`    | The user's `wf_…` Wayfinder API key. Picked up automatically by config priority below. |
| `OPENCODE_INSTANCE_ID` | The Wayfinder Shells identifier for this runtime. Useful for logs / diagnostics.       |

Config priority: `Constructor parameter > config.json > WAYFINDER_API_KEY env var`.

## Messaging the user (Shells instances only)

If you detected a Wayfinder Shells instance in "First-Time Setup" (health probe at `http://localhost:4096/global/health` returned `healthy: true`), you may email the owner to report completed work, surface decisions that need them, or flag anything you can't resolve. The backend only delivers when `email_verified` is true on the user, and throttles to **4 emails / user / day** — budget your sends accordingly. The `message` field is rendered as Markdown (headings, lists, code blocks, tables, links) into a themed HTML email, so format it nicely.

**MCP tool:**
```
notify(
  title="Rebalance complete",
  message="Moved **50 USDC** from Aave → Morpho.\n\n- tx: 0x…\n- new APY: 7.4%",
)
```

**Python client:**
```python
from wayfinder_paths.core.clients.NotifyClient import NOTIFY_CLIENT

await NOTIFY_CLIENT.notify(
    title="Rebalance complete",
    message="Moved **50 USDC** from Aave → Morpho.\n\n- tx: 0x…\n- new APY: 7.4%",
)
```

Both POST to `/api/v1/opencode/notify/` on vault-backend with your `WAYFINDER_API_KEY`. Limits: title ≤ 200 chars, message ≤ 20 000 chars.

## Frontend Context (reading UI state + drawing on charts)

If you detected a Wayfinder Shells instance, you can read what the user is viewing and project overlays (price lines, markers, series) onto their chart in real-time.

**MCP tools:**

| Tool | Args | Description |
|------|------|-------------|
| `get_frontend_context` | (none) | Read current chart context + all projections |
| `add_chart_projection` | `chart_id`, `type`, `config` | Add overlay to a chart |
| `remove_chart_projection` | `chart_id`, `projection_id` | Remove a specific overlay |
| `clear_chart_projections` | `chart_id` | Remove all overlays from a chart |

**Typical flow:**
1. Call `get_frontend_context` → returns `{frontend_context: {chart: {id: "hl-perp-BTC", market_id: "BTC", market_type: "hl-perp", interval: "1m"}}, sdk_projection: {...}}`
2. Read `chart_id` from `frontend_context.chart.id` → `"hl-perp-BTC"`
3. Call `add_chart_projection` with `chart_id="hl-perp-BTC"`, `type="horizontal_line"`, `config={"price": 73500, "color": "#ef4444", "label": "Support"}`
4. Line appears on the user's chart in real-time

**Projection types:**

| type | config |
|------|--------|
| `horizontal_line` | `price`, `color?`, `label?` |
| `marker` | `time` (unix sec), `position` (aboveBar/belowBar), `shape` (circle/arrowUp/arrowDown), `color?`, `label?` |
| `line_series` | `data: [{time, value}]`, `color?`, `label?`, `line_width?` |

**Python client:**
```python
from wayfinder_paths.core.clients.InstanceStateClient import INSTANCE_STATE_CLIENT

state = await INSTANCE_STATE_CLIENT.get_state()
chart_id = state["frontend_context"]["chart"]["id"]

await INSTANCE_STATE_CLIENT.add_projection(chart_id, {
    "type": "horizontal_line",
    "config": {"price": 73500, "color": "#ef4444", "label": "Support"},
})
```

Projections are scoped per chart — switching markets shows only that chart's projections. The backend is type-agnostic; new projection types only need a frontend renderer.

## Scheduled Jobs (backend sync)

On Wayfinder Shells instances (`OPENCODE_INSTANCE_ID` set), the runner daemon automatically syncs job and run state to vault-backend. This happens transparently — no agent action needed.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WayfinderFoundation/wayfinder-paths-sdk](https://github.com/WayfinderFoundation/wayfinder-paths-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
