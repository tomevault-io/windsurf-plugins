---
trigger: always_on
description: Let an AI agent (Claude Code / Codex) directly control the KLayout layout
---

# klink — AI-native control plane for KLayout

Let an AI agent (Claude Code / Codex) directly control the KLayout layout
editor: draw shapes, manage cells/layers, place instances, run pya macros, and
generate layouts with gdsfactory.

## Architecture

```
AI agent
├── MCP: klink-mcp           ← klink RPCs exposed as MCP tools
├── Skill: klayout            ← RPC + pya macro domain knowledge
└── Skill: klayout-gdsfactory ← gdsfactory → KLayout bridge
       │
       ▼
KLayout (klink_plugin)        ← port 8765 (RPC) + 8082 (klive-compat)
```

Do not rely on a hard-coded RPC count. Query the live server:

```python
from klink import KLinkClient

with KLinkClient().connect() as c:
    print([m["name"] for m in c.methods()["methods"]])
```

MCP tools are organized along two orthogonal axes (`klink/mcp/catalog.py`):
**intent/capability** (`read`/`write`/`verify`/`escape`/`all`) and
**domain/area** (`device_photonics`, `routing_backends`, …). `tools/list`
always advertises every tool; call **`klink.find_tools`** to navigate them
(no args → domain index; `domain=<token>` → that area's tools; `query=<keywords>`
→ ranked matches). `--profile` accepts both axes — default
`read,write,verify,escape`.

## Setup

Install the package (`pip install klayout-klink`) and the KLayout salt plugin,
then configure your agent's MCP server:

```json
{
  "mcpServers": {
    "klayout": {
      "command": "<python-that-has-klink>",
      "args": ["-m", "klink.mcp", "--profile", "read,write,verify,escape",
               "--session-id", "project-klink"],
      "env": {
        "KLINK_CONTEXT_ROOT": "<your-project>/.klink/sessions"
      }
    }
  }
}
```

Interpreter rule: optional libraries (gdsfactory, klayout, numpy, …) must live
in the SAME Python that runs `klink.mcp`. klink does not bundle them — install
the one a feature needs yourself; the tool error names it. `klink.status`
reports `interpreter` and `capabilities` so you can verify.

## Environment

- klink core is pure Python; its only runtime dependencies are its own two Rust
  kernels (prebuilt wheels, byte-parity with the pure-Python reference), so
  `pip install klayout-klink` brings klink + both accelerators. They are
  speed-only (pure-Python fallbacks exist); `pip install --no-deps` gives the
  pure-Python core alone.
- gdsfactory must be in the SAME interpreter that runs `klink.mcp`.
- klink RPC port: 8765; klive-compat port: 8082.
- interaction context memory: `.klink/sessions/<session-id>/interaction_context.jsonl`

## Agent Operating Rules

### Tool design rule

Any tool documented for agents to call follows: one user intention = one call,
state persisted on disk not in agent memory, errors are instructions (carry a
`next_action`), validate-before-mutate. Reference implementation:
`photonics.connect` / `photonics.reroute`.

### klink process purity

`klink/` ships only MECHANISM: the `ProcessProfile`, `ConnectivitySpec`, and
`StackSpec` classes plus the routing/LVS algorithms. It holds ZERO process data
— no hardcoded layers, devices, DRC numbers, or PDK instances. Every
process-specific fact is example- or project-owned: your `pdk.py` (scaffolded by
`klink init`) holds the layers / vias / device library and is passed EXPLICITLY
into the klink APIs.

To build a circuit you WRITE OR RUN an example that imports the process + device
library from your `pdk.py` and passes them EXPLICITLY into the klink APIs — never
bake process/device values into `klink/`. A complete, self-contained reference
that owns its own process (layers / spacing / vias) and a synthetic device is:

```text
examples_klink/public/demos/fit_device_pnr_lvs.py
```

It fits a parametric device PCell from exemplar geometry, places it, runs
detailed routing, and verifies with live LVS — importing only `klink`. Copy it
and edit the numbers for YOUR process.

Agent-facing consequence: the `structdevice.*` MCP tools ship NO process, device
library, or terminal recipe. Called without them they return an INSTRUCTIVE
error (not a crash) naming the exact next step. READ the error's `next_action`
and follow it; do NOT invent a `ProcessProfile` or device library yourself.

### Selection-first layout debugging

Never call `view.screenshot` unless the user explicitly asks for a
screenshot/visual artifact in the current conversation. Prefer structured
geometry state:

```text
selection.get
interaction.selection.latest / recent, when available
shape.query
layout.info
cell.list / cell.tree
layer counts
```

Screenshots are user-requested artifacts only, not agent evidence.

### Interaction context memory

`klink-mcp` keeps session-scoped selection memory outside the KLayout plugin.
Memory is recorded when the user clicks the KLayout `SEND` toolbar action or an
agent calls `selection.send_context`. Use the memory tools whenever the user
refers to context they sent/pinned ("just sent", "this area", "here",
"that one"):

```text
interaction.selection.recent   -> latest stored selections, default latest 5
interaction.selection.latest   -> latest stored selection
interaction.selection.get      -> exact stored id
interaction.selection.label    -> attach label/description to an important id
interaction.context            -> current selection plus recent memory
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [klinkdev2026/klayout-klink](https://github.com/klinkdev2026/klayout-klink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
