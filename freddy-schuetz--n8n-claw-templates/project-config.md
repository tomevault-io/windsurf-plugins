---
trigger: always_on
description: This file helps AI assistants (Claude Code, Copilot, etc.) and human contributors create and contribute templates.
---

# CLAUDE.md — n8n-claw Templates

This file helps AI assistants (Claude Code, Copilot, etc.) and human contributors create and contribute templates.

## What is this repo?

A template catalog for [n8n-claw](https://github.com/freddy-schuetz/n8n-claw). Each template is a pre-built MCP server that can be installed via chat command. For the main project architecture, see the [n8n-claw CLAUDE.md](https://github.com/freddy-schuetz/n8n-claw/blob/main/CLAUDE.md).

## Template Structure

Templates come in two shapes — `native` and `bridge`.

**Native** (`type: "native"`) — n8n implements the tool logic itself. 3 files:

```
templates/
  index.json                    <- catalog (one entry per template)
  {template-id}/
    manifest.json               <- metadata (name, tools, credentials)
    workflow.json               <- n8n workflow bundle (sub + server)
```

**Bridge** (`type: "bridge"`) — registers an existing external MCP server (Streamable HTTP) into `mcp_registry`. **No `workflow.json`** — just `manifest.json` + `index.json` entry. Supported since n8n-claw v1.3.0. See [templates/TEMPLATE_EXAMPLE.md#bridge-templates](templates/TEMPLATE_EXAMPLE.md#bridge-templates) for the manifest schema (`bridge.mcp_url`, `auth_type`, `auth_token_required`, …) and [templates/deepwiki/](templates/deepwiki/) as a working no-auth example.

The rest of this document covers the **native** path. If you are shipping a bridge template, stop here and follow the `TEMPLATE_EXAMPLE.md` section above.

## Two-Workflow Pattern

Every **native** template contains **two workflows** in `workflow.json`:

1. **`sub`** — Sub-workflow with actual tool logic (`executeWorkflowTrigger` -> `Code` node)
2. **`server`** — MCP server that exposes the tool (`mcpTrigger` -> `toolWorkflow`)

**Why two workflows?** n8n's API ignores `specifyInputSchema` when creating workflows via API. The `toolWorkflow` + sub-workflow pattern avoids this bug — parameters arrive via `$json.param` which always works.

The Library Manager imports `sub` first, gets its ID, patches `REPLACE_SUB_WORKFLOW_ID` in `server`, then imports `server`.

---

## Valid Categories

Use one of these existing categories. If none fit, open an issue to propose a new one.

`analytics` · `cms` · `communication` · `creativity` · `e-commerce` · `entertainment` · `finance` · `knowledge` · `language` · `maps` · `marketing` · `meetings` · `network` · `news` · `productivity` · `reference` · `smart-home` · `tourism` · `transport` · `utilities` · `weather`

---

## Naming Conventions

| What | Convention | Example |
|------|-----------|---------|
| Template ID | `lowercase-hyphens` | `weather-openmeteo` |
| Template directory | Same as ID | `templates/weather-openmeteo/` |
| Server workflow name | `MCP: Display Name` | `MCP: Weather OpenMeteo` |
| Sub-workflow name | `MCP Sub: Display Name` | `MCP Sub: Weather OpenMeteo` |
| Tool names | `snake_case` | `get_forecast`, `list_contacts` |
| Credential keys | `lowercase_underscore` | `seafile_token`, `lexware_api_token` |

**Consistency rule:** The template ID must match across: directory name, `manifest.json` `id`, `index.json` `id`, and the MCP trigger `path`. Tool names must match across: `manifest.json` `tools[].name`, the toolWorkflow node `name` field, and the `connections` key.

---

## Critical Rules

### HTTP Requests
```javascript
// CORRECT — use helpers (no $)
const data = await helpers.httpRequest({ method: 'GET', url: '...' });

// WRONG — $helpers is undefined in Code node v2
const data = await $helpers.httpRequest({ method: 'GET', url: '...' });
```

### Parameter Passing

In the **sub-workflow** Code node, parameters arrive via `$input`:
```javascript
const input = $input.first().json;
const city = input.city || 'Berlin';
```

In the **server workflow**, parameters are extracted from the user's message via `$fromAI()`:
```javascript
"city": "={{ $fromAI('city', 'City name', 'string') }}"
```

### Required vs Optional Parameters

**Problem:** When a parameter is marked as `"required": true` in the toolWorkflow schema, the MCP server exposes it as required in `tools/list`. If the AI agent doesn't provide it, the tool call fails — even if the sub-workflow code handles missing values with defaults.

**Rule:** Only mark a parameter as `"required": true` if the tool genuinely cannot work without it. Parameters with defaults in the Code node should be `"required": false`.

**Example — wrong:**
```json
"schema": [
  { "id": "action", "type": "string", "required": true },
  { "id": "limit", "type": "string", "required": true }
]
```
Here `limit` has a default in the Code node (`const lim = parseInt(input.limit) || 10`), so marking it required forces the AI to always provide it — even when the default is fine.

**Example — correct:**
```json
"schema": [
  { "id": "action", "type": "string", "required": true },
  { "id": "limit", "type": "string", "required": false }
]
```

**Safety net:** The MCP Client in n8n-claw auto-fills missing required params with empty strings via `tools/list` schema inspection. But this is a fallback — templates should set `required` correctly in the first place.

### Credential Fetching Pattern


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [freddy-schuetz/n8n-claw-templates](https://github.com/freddy-schuetz/n8n-claw-templates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
