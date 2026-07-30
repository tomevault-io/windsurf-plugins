---
trigger: always_on
description: **Analysis Date:** 2026-03-27
---

# Coding Conventions

**Analysis Date:** 2026-03-27

## Naming Patterns

**Files:**
- Executable server entrypoints use hyphenated script names in the repository root: `meraki-mcp.py`, `meraki-mcp-dynamic.py`.
- Helper scripts use snake_case with `.py` suffix: `inspect_tools.py`.
- Documentation files use uppercase or uppercase-hyphen Markdown names: `README.md`, `README-DYNAMIC.md`, `INSTALL.md`, `OPTIMIZATIONS.md`.

**Functions:**
- Manual MCP tool wrappers in `meraki-mcp.py` use snake_case names such as `get_networks`, `update_wireless_ssid`, and `create_action_batch`.
- Dynamic MCP convenience tools in `meraki-mcp-dynamic.py` mirror Meraki SDK method names in camelCase, such as `getOrganizations`, `getNetworkClients`, and `updateDeviceSwitchPort`.
- Internal helpers are prefixed with `_` when not exposed as tools, such as `_build_kwargs` in `meraki-mcp.py` and `_call_meraki_method_internal` in `meraki-mcp-dynamic.py`.

**Variables:**
- Module-level configuration constants are uppercase and loaded from the environment in `meraki-mcp.py`, `meraki-mcp-dynamic.py`, and `inspect_tools.py`: `MERAKI_API_KEY`, `MERAKI_ORG_ID`, `MCP_TRANSPORT`, `CACHE_TTL_SECONDS`.
- Local Python variables use snake_case, even when they are later translated to Meraki SDK camelCase parameters, for example `org_id`, `network_id`, `device_policy`, and `global_bandwidth_limits` in `meraki-mcp.py`.
- Temporary request dictionaries are commonly named `params`, `kwargs`, `update_dict`, `rules_dict`, or `response_data` in `meraki-mcp.py` and `meraki-mcp-dynamic.py`.

**Types:**
- Pydantic models in `meraki-mcp.py` use PascalCase and `Schema` suffixes where the model represents request payloads: `SsidUpdateSchema`, `DeviceUpdateSchema`, `ActionBatchSchema`.
- Support models also use PascalCase: `Dot11wSettings`, `FirewallRule`, `SimpleCache`.
- Type hints rely on built-in generics and `typing` names in both servers, for example `list[str]`, `Dict[str, Any]`, and `Optional[bool]`.

## Code Style

**Formatting:**
- No formatter configuration is present. `pyproject.toml` only defines project metadata and runtime dependencies; there is no Ruff, Black, isort, or autopep8 config.
- Code follows 4-space indentation and keeps one top-level declaration per block in `meraki-mcp.py`, `meraki-mcp-dynamic.py`, and `inspect_tools.py`.
- Tool responses are consistently serialized with `json.dumps(..., indent=2)` in `meraki-mcp.py` and `meraki-mcp-dynamic.py`.
- Section banners made of `###################` are used to break up large single-file modules in `meraki-mcp.py` and `meraki-mcp-dynamic.py`.

**Linting:**
- No lint configuration is detected in the repository root. Files such as `.ruff.toml`, `ruff.toml`, `.flake8`, `setup.cfg`, and `tox.ini` are not present.
- Style consistency is maintained manually. When editing `meraki-mcp.py` or `meraki-mcp-dynamic.py`, match existing spacing, banner comments, and `json.dumps(..., indent=2)` output style.

## Import Organization

**Order:**
1. Standard library imports appear first in `meraki-mcp.py`, `meraki-mcp-dynamic.py`, and `inspect_tools.py`, including `os`, `sys`, `json`, `asyncio`, `functools`, `inspect`, `hashlib`, and `threading`.
2. Third-party imports follow, including `meraki`, `pydantic`, `mcp.server.fastmcp.FastMCP`, and `dotenv.load_dotenv`.
3. There are no local package imports because the repository is a flat script layout without a `src/` package.

**Path Aliases:**
- Not used. All imports are direct module imports from installed dependencies or Python standard library modules.

## Error Handling

**Patterns:**
- Configuration failures are treated as fatal at import time. Both `meraki-mcp.py` and `meraki-mcp-dynamic.py` check `MERAKI_API_KEY`, print a message to `stderr`, and call `sys.exit(1)` if it is missing.
- The dynamic server centralizes runtime error handling in `_call_meraki_method_internal` inside `meraki-mcp-dynamic.py`. It catches `meraki.exceptions.APIError`, `TypeError`, and generic `Exception`, then returns a JSON string with `error`, `message`, and related metadata instead of raising.
- The manual server in `meraki-mcp.py` usually lets SDK exceptions bubble out. Most tool functions are thin wrappers with no `try/except`, so failures depend on FastMCP and the Meraki SDK to surface errors.
- Invalid or unsafe file-cache access is normalized to JSON error payloads in `get_cached_response` and `list_cached_responses` in `meraki-mcp-dynamic.py`.

## Logging

**Framework:** `print` to `stderr`

**Patterns:**
- Meraki SDK logging is intentionally suppressed with `suppress_logging=True` in `meraki-mcp.py`, `meraki-mcp-dynamic.py`, and `inspect_tools.py`.
- Startup and configuration messages are emitted with `print(..., file=sys.stderr)` in `meraki-mcp.py`, `meraki-mcp-dynamic.py`, and `entrypoint.sh`.
- The Python `logging` module is not used anywhere in the repository.

## Comments

**When to Comment:**
- Use short banner comments to separate major domains or systems, following the existing style in `meraki-mcp.py` and `meraki-mcp-dynamic.py`.
- Add brief inline comments only when they explain operational intent, such as cache invalidation, read-only behavior, or transport selection.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CiscoDevNet/meraki-magic-mcp-community](https://github.com/CiscoDevNet/meraki-magic-mcp-community) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
