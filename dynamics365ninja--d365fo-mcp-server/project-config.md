---
trigger: always_on
description: <!-- Mirrors rules from xpp_system_instructions MCP prompt (src/prompts/systemInstructions.ts). Keep in sync. -->
---

# D365 Finance & Operations X++ Development

<!-- Mirrors rules from xpp_system_instructions MCP prompt (src/prompts/systemInstructions.ts). Keep in sync. -->

This workspace contains D365FO code. **Always use the specialized MCP tools** — backed by a pre-indexed symbol database with hundreds of thousands of D365FO objects. Built-in file/search tools do not understand X++ syntax or AOT structure.

---

## 🚨 TERMINAL PROHIBITION

**PowerShell / any terminal command WILL HANG in this workspace.** This applies in VS Code, VS 2022, VS 2026.

- **NEVER** run `run_in_terminal`, Developer PowerShell, or any shell command
- **NEVER** use terminal as fallback when an MCP tool fails — STOP and report the error
- If a tool parameter "seems missing" — re-read the schema; it IS present

## 🔓 WHEN MCP IS OPTIONAL

MCP rules apply **only to D365FO objects** (`.xml`/`.xpp`, AOT objects, labels, `.rnrproj`).

**Use built-in tools freely for:** `*.cs`, `*.json`, `*.yml`, `*.md`, `*.config`, `*.csproj`, `*.sln`, plain text, or when user says "skip MCP" / "manual mode".

- **`.rnrproj`** = D365FO project → managed by MCP (`addToProject=true`). NEVER edit directly.
- **`.csproj`** = C# project → use built-in tools.

## � HOW READS ARE RESOLVED (read-path policy)

Info tools (`get_class_info`, `get_table_info`, `get_form_info`, `get_view_info`, `get_query_info`, `get_report_info`, `get_table_extension_info`, `find_coc_extensions`, `analyze_extension_points`) resolve data in this order:

1. **C# bridge** — live `IMetadataProvider` from the running D365FO instance. Authoritative when available.
2. **SQLite symbol index** — pre-built mirror. Used when the bridge is offline (Azure, write-only mode, build agents).
3. **Filesystem parse** — last resort for objects created in the current session and not yet indexed. Scanner has a 3 s budget, 30 s result cache, and can be disabled in production with `D365FO_DISABLE_FS_FALLBACK=true`.

You never need to pick the source manually — just call the tool. If you see `⚠️ Served from symbol index` or `⚠️ Not yet in bridge metadata`, the bridge was unavailable and the tool already fell back.

## 🛡️ WRITE-PATH SAFETY

All write operations (`modify_d365fo_file`, `create_d365fo_file`) only accept paths that live under a configured `PackagesLocalDirectory/<Package>/<Model>/Ax<Type>/<Name>.xml`. Arbitrary paths are rejected.

---

## �🔌 MANDATORY FIRST CHECK

**Call `get_workspace_info()` before doing anything.**

| Response | Action |
|----------|--------|
| Call fails | STOP. Tell user MCP server is not connected. Offer: start server (A) or continue with built-in tools (B). Wait for answer. |
| "not available in read-only mode" | Azure mode. Ask user for model name explicitly. Do NOT infer from search results. |
| `⛔ CONFIGURATION PROBLEM` | STOP. Relay message. Wait for user. |
| `✅ Configuration looks valid` | Note model name. Use it for all create/modify calls. Proceed. |

If you encounter `MyModel`/`MyPackage` placeholder mid-task — STOP and notify user.

## ✏️ EDITING D365FO FILES

| Action | Tool |
|--------|------|
| Edit existing objects | `modify_d365fo_file()` — methods, fields, indexes, relations, field-groups, controls, properties |
| Create new objects | `create_d365fo_file()` |
| Search | `search()`, `batch_search()` |
| Read objects | `get_class_info()`, `get_table_info()`, `get_form_info()`, `get_report_info()` |
| Verify project | `verify_d365fo_project()` |
| Build/BP/Sync/Test | `build_d365fo_project()`, `run_bp_check()`, `trigger_db_sync()`, `run_systest_class()` |

**NEVER use** `replace_string_in_file`, `edit_file`, `create_file`, `read_file`, `grep_search`, `code_search` on D365FO `.xml`/`.xpp` files.

**`overwrite=true` on `create_d365fo_file`** — ONLY for full XML replacement. NEVER for incremental changes (add-field, add-field-group, etc.) → use `modify_d365fo_file`.

**`dryRun=true`** — use for multi-line or public API changes. Show diff to user. Wait for confirmation before applying.

### ⛔ Escalating-workarounds anti-pattern — STOP at step 0

If `modify_d365fo_file` is the correct tool but you feel tempted to try something else, you are wrong. STOP.
```
WRONG SPIRAL (each step is MORE wrong):
 Step 1: "I'll use replace_string_in_file to patch the XML"
 Step 2: "replace failed — I'll try a different approach"
 Step 3: "I'll read the file with PowerShell first, then overwrite"
 Step 4: "Terminal returns no output — I'll add Write-Output"
 Step 5: "I'll use create_d365fo_file with overwrite=true"

CORRECT (always, immediately):
 modify_d365fo_file(operation="add-field-group" | "add-field" | "add-method" | …)
```
If `modify_d365fo_file` itself errors — STOP and report to user. Do NOT try PowerShell.

### `modify_d365fo_file` — full operation inventory

| Category | Operations |
|----------|------------|
| Methods | `add-method`, `remove-method`, `replace-code` |
| Fields | `add-field`, `modify-field`, `rename-field`, `replace-all-fields`, `remove-field` |
| Indexes | `add-index`, `remove-index` |
| Relations | `add-relation`, `remove-relation` |
| Field groups | `add-field-group`, `remove-field-group`, `add-field-to-field-group` |
| Table-ext | `add-field-modification` (override base-table field label/mandatory) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dynamics365ninja/d365fo-mcp-server](https://github.com/dynamics365ninja/d365fo-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
