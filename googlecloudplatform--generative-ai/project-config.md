---
trigger: always_on
description: > **Purpose**: Project-specific knowledge for AI coding agents (Antigravity, Cursor, Copilot, etc.)
---

# AGENTS.md — AI Agent Development Guide for GE Demo Generator

> **Purpose**: Project-specific knowledge for AI coding agents (Antigravity, Cursor, Copilot, etc.)
> and humans working on this sample.

---

## 1. Architecture

```
app/Code.gs (Apps Script backend, ~5,300 lines)
  ├─ UI server + Gemini calls (demo planning, data synthesis)
  └─ generateSetupScript(): emits a bash setup script that
       1. fetches agent_template/ from this repo at a PINNED ref
          (sparse git checkout of TEMPLATE_SUBDIR at TEMPLATE_REF)
       2. copies the static Python/JSON files into the demo project
       3. writes small per-demo config next to them:
            .env                      (scalars + feature flags)
            adk_agent/app/generated_instruction.md
            adk_agent/app/mcp_config.json
       4. builds the container and deploys to Cloud Run

agent_template/ (real, testable files — fetched at setup run time)
  ├─ adk_agent/app/agent.py            ADK agent (reads env + config files)
  ├─ adk_agent/app/tools.py            toolsets (env-gated feature blocks)
  ├─ adk_agent/app/fast_api_app.py     A2A/FastAPI runtime
  ├─ adk_agent/app/part_converters.py  A2UI part conversion
  ├─ adk_agent/app/examples/0.8/*.json A2UI few-shot examples
  ├─ managed_agent/                    Managed Agent provisioning helpers
  │                                    (create_managed_agent.py, warmup_managed_agent.py)
  ├─ demo_skills/                      Deliverable craft skills mounted into the
  │                                    Managed Agent sandbox (SKILL.md packs)
  └─ viewer_app/                       Firestore data viewer (Cloud Run Functions)
```

Per-demo variation is passed at run time, never baked into the Python:

| Mechanism | Contents |
|---|---|
| Environment variables | `DEMO_DATASET`, `FS_COLLECTION`, `REFERENCE_DATE`, `PUBLIC_DATASET_ID`, `ENABLE_WORKSPACE_MCP`, `ENABLE_COMPUTER_USE`, `ENABLE_MANAGED_AGENT`, `ENABLE_WORKSPACE_AUTH`, `MANAGED_AGENT_ID`, `MANAGED_AGENT_SKILLS_SOURCE` (plus the pre-existing `DEMO_ID`, `DATA_VIEWER_URL`, …) |
| `generated_instruction.md` | The Gemini-generated system instruction for the demo |
| `mcp_config.json` | Imported MCP servers (name, entrypoint, port, auth type) |
| Placeholders substituted by the setup script | `[CURRENCY]` in the example JSONs; `__GE_FS_COLLECTION__` / `__GE_DASH_TITLE__` / `__GE_DASH_DESC__` in `viewer_app/main.py` |

## 2. Editing rules

### 2.1 agent_template/ — plain Python and JSON

Edit directly. No escaping rules apply. Validate with:

```bash
python3 validate_examples.py   # json.loads every example + py_compile every .py
```

Feature-dependent code is gated at run time, not generation time:

```python
if os.environ.get("ENABLE_WORKSPACE_MCP") == "1":
    ...workspace toolsets...
```

Keep that pattern — do not reintroduce generation-time code selection.

### 2.2 Code.gs — remaining generated bash

`generateSetupScript` still emits bash (BigQuery/Firestore provisioning,
Dockerfile assembly, deployment). Inside those JS template literals:

- Emit a literal bash `${VAR}` as `\${VAR}`; a literal backtick as `` \` ``.
- `\` + newline inside a JS template literal is a line continuation (the
  newline disappears from the output). Use it only intentionally.
- Quoted heredocs (`cat <<'X'`) pass content through verbatim; unquoted
  heredocs (`cat <<X`) expand `$VAR` at run time.

### 2.3 ADK instruction template engine hazard (applies to agent.py)

ADK's `inject_session_state()` scans the agent `instruction` with the regex
`r'{+[^{}]*}+'` before every LLM request and raises
`KeyError: 'Context variable not found: ...'` for unknown names — crashing the
request. `{var}`, `{{var}}`, and `{{{var}}}` are all matched; double-bracing
does NOT escape.

- WRONG in instruction text: `.../collection/{document_id}`, `/form/item_{i}_name`
- RIGHT: `.../collection/<document_id>`, `/form/item_i_name` (or `[i]`)

`[BRACKET]` and `<angle_bracket>` notations are safe. This is why the
instruction pipeline uses `[PROJECT_ID]`-style tokens with `str.replace`.

## 3. Managed Autonomous Agent (`enableManagedAgent`)

Optional feature (default ON in the UI) that provisions a Pre-GA **Managed
Agents API** agent (Antigravity harness, location `global` only) the ADK agent
can delegate long-running autonomous work to over the **Interactions API**
(REST + SSE via httpx — intentionally no new pinned dependency).

- **Flag thread**: `index.html` toggle → `generateSetupScript` → PHASE A
  (right after the dashboards bucket exists: skills upload to
  `gs://<dash-bucket>/skills`, `managed_agent_instruction.txt` heredoc,
  `create_managed_agent.py start`) → `.env` + Cloud Run env
  (`ENABLE_MANAGED_AGENT`, `MANAGED_AGENT_ID`, `MANAGED_AGENT_SKILLS_SOURCE`)
  → env-gated blocks in `tools.py` / `agent.py` / `fast_api_app.py` → PHASE B
  (after Cloud Run deploy + GE registration: `create_managed_agent.py wait`
  polls readiness, `warmup_managed_agent.py` stores the environment id in
  Firestore `<demo>_managed_agent_state/current`). The A/B split hides the
  ~8-10 min agent creation behind the rest of the setup.
- **`enableWorkspaceAuth` (auth-only mode)**: sets up the GE OAuth

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GoogleCloudPlatform/generative-ai](https://github.com/GoogleCloudPlatform/generative-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
