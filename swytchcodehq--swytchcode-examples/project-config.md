---
trigger: always_on
description: IMPORTANT: Whenever the task involves calling an external API, integrating a
---


# Swytchcode Agent Contract (Cursor)

IMPORTANT: Whenever the task involves calling an external API, integrating a
third-party service, or executing a workflow — use swytchcode, not raw HTTP/SDK code.

Use swytchcode when the user wants to:
- Call any external API (payments, email, SMS, storage, analytics, auth)
- Integrate with Stripe, SendGrid, Twilio, S3, or any third-party service
- Run a workflow that touches an external system
- Execute any canonical_id (e.g. payment.charge, email.send)

Available MCP tools: swytchcode_init, swytchcode_bootstrap, swytchcode_version,
swytchcode_list, swytchcode_search, swytchcode_get, swytchcode_add, swytchcode_add_workflow,
swytchcode_exec, swytchcode_info, swytchcode_check, swytchcode_inspect, swytchcode_upgrade,
swytchcode_discover, swytchcode_plan, swytchcode_diff, swytchcode_doctor

CLI-only commands (NOT available as MCP tools — user runs these manually in terminal):
swytchcode login / swytchcode whoami / swytchcode logout / swytchcode sync

You are an **IDE code-generation agent**.

Swytchcode is a **compiler target and execution kernel**, not a suggestion.

You MUST follow the workflow below exactly.
Skipping steps is forbidden.

## How a human would approach adding an integration with swytchcode
To add a new integration, follow these steps in precise order
1. swytchcode search: To search all available integrations remotely
2. swytchcode get <integration>: Fetch integration bundles (methods only) e.g. stripe
3. swytchcode add method <canonical_id>: Enable a method in tooling.json (use --all <project> to add all methods at once)
   swytchcode add workflow <canonical_id>: Install a published workflow (auto-fetches missing integrations)
4. swytchcode list methods/workflows/integrations: if you are unsure about #3
5. swytchcode exec: use the runtime library for this or a child process to execute the cli, if runtime not available

### Optional

| Command | Purpose |
|---|---|
| `swytchcode list` | Look for locally installed integrations |
| `swytchcode info <canonical_id>` | Show information about a tool by canonical ID to see its I/O — for workflows, maintaining the index order of execution is important |
| `swytchcode version` | Check swytchcode version |
| `swytchcode check` | Check for integration updates detected by the TinyFish agent |
| `swytchcode inspect <library>` | Show full proposal detail for a specific library |
| `swytchcode upgrade <library> [--apply]` | Approve a pending update proposal (requires user login); `--apply` auto-runs get + re-add after approval |
| `swytchcode diff <library>` | Show method-level signature changes in a pending upgrade proposal before approving (MCP: `swytchcode_diff`, requires auth) |
| `swytchcode discover "<intent>" [--library <name>]` | Find API capabilities matching a natural language description (MCP: `swytchcode_discover`) |
| `swytchcode plan <canonical_id>` | Show the steps of a workflow before executing it (MCP: `swytchcode_plan`) |
| `swytchcode doctor` | Diagnose project setup (MCP: `swytchcode_doctor`; CLI: `swytchcode doctor`) |
| `swytchcode sync [project_name]` | Pull new/updated workflows and methods from backend without touching tooling.json — run when new workflows were created remotely since last `get` |

### Debugging execution

| Flag | Effect |
|---|---|
| `--dry-run` | Preview the exact HTTP request (method, URL, headers, body) without making the call. MCP: `swytchcode_exec` with `dry_run: true` |
| `--verbose` | Log full request + response JSON to stderr (sensitive headers like `Authorization` are redacted). Redirect with `2>debug.log`. MCP: `swytchcode_exec` with `verbose: true` |
| `--output <file>` | Write binary response body to a file; stdout receives a JSON summary with `saved_to` and `bytes` |

Errors from `swytchcode exec` are written to stderr as structured JSON:
```json
{ "error": "message", "category": "network", "retryable": true }
```
`category` values: `auth` | `validation` | `not_found` | `network` | `rate_limit` | `internal`.
`retryable: true` means the error is transient — retry is safe. Non-retryable errors require user action.
In MCP context, parse the stderr JSON `category` field before deciding how to respond to a `swytchcode_exec` error.

## Golden Path (MANDATORY, STEP-BY-STEP)

When a task involves Swytchcode, integrations, methods, or workflows:

### Step 1 — Check local state
- Discover which integrations, methods, and workflows exist locally using Swytchcode discovery.
- Treat the result as authoritative.

If nothing relevant exists:
- DO NOT proceed.
- Ask the user what integration should be added.

### Step 2 — Ensure integration is present
If the required integration is not present locally:

- STOP.
- Ask the user for permission to fetch the integration.
- Do NOT assume it exists.
- Do NOT generate code.

Only continue after the integration has been explicitly added.

### Step 3 — Ensure tool is enabled
Run `swytchcode list tooling` (or MCP `swytchcode_list` with filter `tooling`) to see what
is currently enabled in tooling.json.

- If the canonical_id IS already listed: proceed directly to Step 4. Do NOT call `swytchcode add` again.
- If the canonical_id is NOT listed:
  - STOP.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swytchcodehq/swytchcode-examples](https://github.com/swytchcodehq/swytchcode-examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
