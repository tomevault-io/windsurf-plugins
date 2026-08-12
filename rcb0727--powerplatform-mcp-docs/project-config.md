---
trigger: always_on
description: **ALWAYS follow this workflow when users want to create, build, or modify a Power Automate flow:**
---

# Power Platform MCP Server — Claude Code Instructions

## The 6-Phase Flow Workflow

**ALWAYS follow this workflow when users want to create, build, or modify a Power Automate flow:**

### Phase 1: PLAN
1. Call `plan_flow` with the user's description
2. Present ALL clarifying questions to the user (don't skip any)
3. Wait for answers before proceeding
4. If answers are incomplete, ask follow-up questions

### Phase 2: REVIEW
1. Show the user what will be built:
   - Trigger type and timing
   - Actions in order
   - Required connections
2. Confirm before creating
3. Warn about Premium connectors if detected

### Phase 3: VALIDATE
1. For complex flows, use `validate_flow` on the definition
2. Warn about missing error handling
3. Suggest best practices if violations detected

### Phase 4: CREATE
1. Create the flow with `create_flow` or `build_flow`
2. Note the flow ID for testing
3. Remind user the flow is created but stopped by default

### Phase 5: TEST
1. **ALWAYS test after creating or modifying a flow**
2. Use `test_flow` for guided testing with diagnostics
3. Or use `run_flow` for quick execution
4. Check results immediately

### Phase 6: DEBUG (if needed)
1. If test fails, call `diagnose_flow` immediately
2. Show user the error category and suggested fix
3. Offer to apply the fix
4. Re-test after fixing
5. Repeat until flow succeeds

## Canvas App Source Workflow (Preview)

When asked to create or modify a Canvas app, use an existing blank/editable app shell and follow this sequence:

1. Call `connect_canvas_authoring` with the Power Apps Studio Designer URL. Do not guess account, tenant, or authentication overrides.
2. Call `sync_canvas_source` into a dedicated absolute directory and inspect all returned `.pa.yaml` files. The Studio tab must remain open with coauthoring enabled.
3. Before adding controls, call `list_canvas_controls`, then `describe_canvas_control` for every control type used. Likewise discover APIs and data-source schemas instead of guessing them.
4. Write `App.pa.yaml` plus one `.pa.yaml` file per screen. Use `expectedSha256` for every overwrite or delete so concurrent edits cannot be lost silently.
5. Call `compile_canvas_source` with `confirm=true`, fix every reported file/line error, and repeat until clean. Compilation changes the live Studio draft.
6. Re-sync or verify in Studio, then ask the maker to test, save, and publish. A clean compile does not prove runtime behavior, layout, accessibility, data authentication, save, or publish.

## Tool Reference

<!-- TOOLS-TABLE:BEGIN — generated from the tool registry and synced here at each release; do not edit by hand -->
## Available Tools (228 total)

> Every tool the server exposes, grouped by service. All 228 are listed here.

<details>
<summary><strong>Setup & Authentication</strong> (1 tools)</summary>

| Tool | Description |
|------|-------------|
| `sign_in` | Sign in to Microsoft Power Platform when other tools report missing credentials. Starts Microsoft's device-code sign-in and returns a code plus a microsoft.com link for the user to complete in their own browser (MFA included) — no terminal needed. Call sign_in again after the user says they've finished to confirm. This tool only displays the code; it never asks for or accepts passwords. |

</details>

<details>
<summary><strong>Core Flow Operations</strong> (15 tools)</summary>

| Tool | Description |
|------|-------------|
| `list_flows` | List Power Automate flows in an environment. Returns flow names, IDs, state (Started/Stopped), last modified dates, and owners. Use scope='shared' to find flows shared with you, scope='owned' for your flows only. Use this to discover what flows exist before getting details or making changes. |
| `get_flow` | Get the complete definition of a Power Automate flow including triggers, actions, connection references, and description. Use list_flows first to find flow IDs. Set format='json' or format='both' to capture the FULL definition including nested actions inside Switch/If/Foreach/Scope (the default 'summary' format only shows top-level actions). |
| `create_flow` | Create a new Power Automate flow. Provide a trigger, actions, and connection references. Use search_connectors and get_action_schema to understand the required parameters for connector actions. The flow is created enabled (Started); use toggle_flow to stop it. |
| `update_flow` | Update an existing Power Automate flow. Three update modes: (1) full replace — pass 'actions' to replace all top-level actions (default; you must include nested actions you want kept), (2) merge — set mergeActions=true to deep-merge only the actions you provide, leaving the rest intact, (3) patch — use patchActions with path keys (e.g. 'If/cases/Default/actions/Compose') for surgical edits with the smallest payload. Use get_flow with format='json' first to see the full nested action tree. |
| `preview_update` | Preview exactly what update_flow would change on a flow — display name, trigger, added/removed/modified actions, and connection references — without writing anything. Run this before update_flow on flows that matter. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rcb0727/powerplatform-mcp-docs](https://github.com/rcb0727/powerplatform-mcp-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
