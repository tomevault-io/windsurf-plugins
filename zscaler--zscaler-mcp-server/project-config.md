---
trigger: always_on
description: 280+ tools for managing the Zscaler Zero Trust Exchange. Services: ZPA, ZIA, ZDX, ZCC, EASM, Z-Insights, ZIdentity, ZTW (Zscaler Workload Segmentation).
---

# Zscaler MCP Server

280+ tools for managing the Zscaler Zero Trust Exchange. Services: ZPA, ZIA, ZDX, ZCC, EASM, Z-Insights, ZIdentity, ZTW (Zscaler Workload Segmentation).

## Tool Naming & Discovery

All tools follow `{service}_{verb}_{resource}` naming: `zia_list_locations`, `zpa_create_access_policy_rule`, `zdx_get_application`. Service prefixes: `zia_`, `zpa_`, `zdx_`, `zcc_`, `easm_`, `zins_`, `zid_`, `ztw_`. Use the prefix to discover tools for a given service.

## Critical Gotchas

- **ZIA requires activation.** After any ZIA create/update/delete, call `zia_activate_configuration()`. Changes are staged until activation. Forgetting this is the #1 source of "my change didn't work" issues.
- **OneAPI is the only supported authentication mode.** All tools authenticate via `zscaler.ZscalerClient` against ZIdentity using the unified `ZSCALER_CLIENT_ID` / `ZSCALER_CLIENT_SECRET` (or `ZSCALER_PRIVATE_KEY`) / `ZSCALER_VANITY_DOMAIN` / `ZSCALER_CUSTOMER_ID` (ZPA only) credentials.
- **ZDX is read-only.** ZDX tools only query data. There are no create/update/delete operations except for deep traces (`zdx_start_deep_trace`).
- **ZDX `since` parameter is in hours**, not timestamps. Default is 2 hours. Example: `since=24` means "last 24 hours."
- **IDs are strings**, even when they look numeric. Always pass IDs as strings.
- **ZPA dependency chain matters.** To onboard an application: create app connector group -> create server group (references connector group) -> create segment group -> create application segment (references server and segment groups) -> create access policy rule. Skipping dependencies causes cryptic 400 errors.
- **ZIA dependency chain for locations.** To onboard a location: create static IP -> create VPN credential (references static IP) -> create location (references VPN credential and static IP). The location won't work without the traffic forwarding prerequisites.
- **ZIA policy-rule updates are PUT, not PATCH.** Every ZIA `update_*_rule` tool maps to a PUT (full replacement). Tools silently backfill `name` and `order` from the existing rule when missing, but other fields omitted from the payload may be reset by the API. Affected: `zia_update_ssl_inspection_rule`, `zia_update_cloud_firewall_dns_rule`, `zia_update_cloud_firewall_ips_rule`, `zia_update_file_type_control_rule`, `zia_update_sandbox_rule`.
- **ZIA cloud-application catalogs are NOT interchangeable.** Shadow IT analytics catalog (`zia_list_shadow_it_apps`) uses friendly names + numeric IDs. Policy-engine catalog (`zia_list_cloud_app_policy`, `zia_list_cloud_app_ssl_policy`) uses canonical `UPPER_SNAKE_CASE` enums. Tools that accept friendly names (SSL inspection, file type control, cloud-app control) auto-resolve them and surface the audit in `_cloud_applications_resolution`.
- **DNS rules use `applications`, not `cloud_applications`.** The `applications` field on DNS rules refers to DNS tunnels and network applications — it is not the cloud-app enum catalog and is not auto-resolved.
- **ZIA Sandbox rules vs sandbox reports.** `zia_*_sandbox_rule` tools manage Sandbox **policy rules**. `zia_get_sandbox_*` tools (quota, report, behavioral_analysis, file_hash_count) are read-only sandbox **reports**. Don't confuse the two.

## Write Operations — Safety Rules

1. **Write tools are disabled by default.** Enable with `--write-tools` flag and an explicit allowlist (wildcards supported). Example: `--write-tools "zpa_create_*,zia_update_*"`.
2. **Always confirm before mutating.** Read operations are safe. Create/update/delete operations modify the live Zscaler environment. Ask the user before executing write operations.
3. **Delete operations require HMAC-SHA256 confirmation.** Destructive actions return a confirmation token that must be passed back to confirm. Controlled by `ZSCALER_MCP_SKIP_CONFIRMATIONS` and `ZSCALER_MCP_CONFIRMATION_TTL`.
4. **Always list/get first** to understand current state before creating or modifying resources.
5. **Pagination:** List tools support `page` and `page_size` parameters. For large tenants, paginate rather than fetching everything.
6. **ZPA policy rule ordering:** New rules are appended at the end by default. Policy rules are evaluated top-to-bottom — order matters for access control.

## ZDX Filtering

ZDX query tools accept optional filters that significantly improve result quality:

- `location_id`: Filter by office/site location
- `department_id`: Filter by department
- `geo_id`: Filter by geolocation
- `since`: Hours to look back (default 2)

Always ask the user for scope before running broad ZDX queries on large tenants.

## Environment

Required env vars (set in `.env`):

- `ZSCALER_CLIENT_ID` — OneAPI client ID from the ZIdentity console
- `ZSCALER_CLIENT_SECRET` — OneAPI client secret (or `ZSCALER_PRIVATE_KEY` for JWT-based auth)
- `ZSCALER_VANITY_DOMAIN` — ZIdentity vanity domain (e.g. `acme.zsapi.net`)
- `ZSCALER_CUSTOMER_ID` — Zscaler customer/tenant ID; required when calling ZPA tools

Optional:

- `ZSCALER_PRIVATE_KEY` — PEM-encoded private key for JWT auth (used in place of `ZSCALER_CLIENT_SECRET`)
- `ZSCALER_CLOUD` — cloud override (e.g. `BETA`, `zscalertwo`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zscaler/zscaler-mcp-server](https://github.com/zscaler/zscaler-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
