---
trigger: always_on
description: - Standalone process — **no imports** from `ada_backend/` or `engine/`.
---


## MCP Server Conventions

- Standalone process — **no imports** from `ada_backend/` or `engine/`.
- Tools are thin wrappers: validate inputs, call `DraftnrunClient` or `SupabaseClient`, format response. No business logic.
- Tool naming: `snake_case` verbs matching the API action (`list_`, `get_`, `create_`, `update_`, `delete_`).
- Auth: `SupabaseProvider` from FastMCP. Tokens are standard Supabase JWTs. MCP server does NOT issue its own tokens.
- Consent page: lives in `back-office/src/pages/oauth/callback.vue`, NOT in the MCP server. Supabase redirects to `SITE_URL + authorization_path` for consent.
- Use `get_access_token()` from `fastmcp.server.dependencies` to get the JWT; `.token` for the raw string, `.claims["sub"]` for user ID.
- Org context: session-scoped via Redis and keyed by the MCP session ID when available. Tools must check for active org before calling org-scoped endpoints (use `require_org_context`). Developer+ operations (create/update/delete/pause/resume cron, OAuth, `create_agent`, knowledge mutations, git sync configure/disconnect) use `require_role`. `trigger_cron` is org-scoped (member+).
- Workflow creation: `create_workflow` creates workflow-type projects. Agent projects use `create_agent`.
- Target-org permissions: tools that accept an explicit `org_id` must validate the caller's role on that target organization, not only on the active org session.
- Component search: `search_components(query)` must reject blank or whitespace-only queries instead of returning the full catalog.
- Error handling: never bare except. Map HTTP errors to descriptive MCP tool error messages via `ToolError`.
- Sentry: `MCPIntegration` is auto-configured. Use `sentry_sdk.logger` for structured logging.
- Response trimming: large payloads (>50KB) are truncated by `DraftnrunClient._trim_response`. Trimming is controllable per-tool via `ToolSpec.trim` (default `True`) and per-call via the `trim` parameter on all HTTP methods. `get_graph` and `list_components` are untrimmed.
- When adding/removing/renaming MCP tools: update `mcp_server/README.md`, the relevant `docs://` resource in `mcp_server/docs.py`, and this rule file in the same diff.
- When changing any MCP behavior, payload, auth, limitation, or guardrail: update the matching `docs://` resource in `mcp_server/docs.py` in the same diff.
- Example UUIDs in `docs.py` must come from code (`uuid.uuid4()`), stored as module constants / placeholders replaced at import — never hand-written hex in prose. See `_GRAPH_DOC_*` in `docs.py`.
- When changing file/document behavior: always update `docs://file-management` and `mcp_server/README.md` in the same diff.
- When adding a new tool domain: create a file in `mcp_server/tools/`, register in `mcp_server/tools/__init__.py`.
- Never reuse IDs, instance UUIDs, source IDs, or graph JSON from another project/org as a shortcut. Re-fetch current state with `get_graph`, `list_components`, `search_components`, `list_sources`, etc.
- Ask for explicit user permission before publish/delete/revoke OAuth or guiding OAuth setup in the web UI.
- Prefer dedicated components over `python_code_runner` / `terminal_command_runner`; keep code tools for small, bounded tasks.
- Prefer dedicated search components for web research when the catalog already provides them.
- Do not describe knowledge documents as original downloadable files or claim MCP has a generic file upload/download API unless that behavior actually exists.
- Factory spec validation at registration time catches unresolved path placeholders, `body_org_key` without org/role scope, and missing roles on role-scoped specs.
- Body fields with `None` values are omitted from the JSON payload (not serialized as JSON `null`).
- Redis fallback is recoverable: after a connection failure, the client retries automatically after 60 s.
- Treat `_truncated` responses as partial data, never as complete truth.
- Tool port configurations: component instances in graph payloads include `port_configurations` (setup modes: `ai_filled`, `user_set`, `deactivated`) and a computed `tool_description`. `tool_description_override` customizes the description shown to the AI. When changing tool port behavior, update `docs://graphs`, `docs://agent-config`, and `mcp_server/README.md`.
- QA custom columns: entry `custom_columns` dicts are keyed by column UUID, not display name. Always use `list_custom_columns` to discover the mapping before writing. See `docs://qa` and `docs://known-quirks`.
- CSV tools: `export_dataset_csv` / `import_dataset_csv` handle dataset round-trips. The export builds CSV from paginated `list_entries` internally (no `graph_runner_id` needed). The import uses multipart file upload to the backend's CSV import endpoint.
- Client methods: `api.get_raw()` returns raw text (for non-JSON endpoints). `api.post_file()` handles multipart uploads. Both are available for custom tools that need non-JSON request/response patterns.
- Org selection must be sequential — parallel calls with `select_organization` race and fail. See `docs://known-quirks`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Scopeo/draftnrun](https://github.com/Scopeo/draftnrun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
