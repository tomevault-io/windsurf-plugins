---
trigger: always_on
description: > **Read before writing any tool, test, or documentation.**
---

# Agent Instructions — HA-MCP-Readonly

> **Read before writing any tool, test, or documentation.**

## Language & Naming

### Mandatory English
- ALL code, comments, docstrings, commit messages, and tool descriptions MUST be in English.
- No Polish, no mixed-language fragments (`np.` → `e.g.`, `Zamiast` → `Instead of`).
- No Polish characters (ą, ę, ś, ć, ń, ó, ł, ż, ź) in source files.

### Generic Names Only
- Use generic, non-culture-specific names in examples and test fixtures:
  - `light.living_room` not `light.salon`
  - `area_id="office"` not `area_id="biuro"`
  - `person.test_user` not `person.pawel`
  - `zone.home`, `zone.work` not culture-specific zone names
- Mock data in `tests/fixtures.py` uses: `living_room`, `office`, `bedroom`

### Tool Descriptions
- First line of `@mcp.tool()` docstring MUST be a complete sentence describing what the tool does.
- NO emoji in tool description first lines.
- NO emoji in API response strings (status labels, messages).
- Every docstring must include `Args` and `Returns` sections.
- Use plain text status labels: `"OK"` not `"✅ OK"`, `"FAILED"` not `"❌ FAILED"`.

### Parameter Descriptions
- Use `e.g.` not `np.` for examples.
- Examples must use generic entity IDs: `light.living_room`, `sensor.temperature`, `person.test_user`.

---

## Test Standards

### Test Hierarchy

| Suite | Location | Runtime | Requires | Run with |
|-------|----------|---------|----------|----------|
| **Unit** | `tests/unit/` | <20s | Nothing | `pytest tests/unit/ -q` |
| **Smoke** | `tests/smoke/` | <5s | REST API (ports 9092/9093) + HA_TOKEN | `pytest tests/smoke/ -q` |
| **Integration** | `tests/integration/` | ~2min | Real HA + HA_TOKEN | `pytest tests/integration/ -q` |
| **E2E** | `tests/e2e/` | ~30s | Real HA + REST API + HA_TOKEN | `pytest tests/e2e/ -q` |

### Test Rules

1. **Unit tests:** Zero I/O, all dependencies mocked via `unittest.mock.patch`. Run without credentials.
2. **Smoke tests:** Direct REST API calls (`requests` library), no MCP wrapper needed. Skip if no `HA_TOKEN`.
3. **Integration tests:** Real HA via MCP wrapper (`MCPWrapper` from `tests/integration/conftest.py`). Skip if no `HA_TOKEN`.
4. **E2E tests:** Full pipeline (context generator) + REST API endpoints. Skip if no `HA_TOKEN`.
5. **Zero hardcoded names** in any test data — use mock fixture values.
6. **Test isolation:** Each test must be independent. Post-rely on shared state or test order.
7. **Skip, don't fail:** All non-unit tests use `pytest.mark.skipif(not HA_TOKEN, ...)`.

### Test Environment

1. Copy `.env.example` to `.env`
2. Fill in `HA_URL` and `HA_TOKEN`
3. `.env` is gitignored — never committed

### Writing Tests for a New Tool

Before writing any tool that calls the HA REST API:

1. **Verify the endpoint** in [official HA REST API docs](https://developers.home-assistant.io/docs/api/rest/)
2. **Test with `curl`** + LLAT on a real HA instance:
   ```bash
   curl -s -H "Authorization: Bearer $HA_TOKEN" "http://HA_IP:8123/the/endpoint"
   ```
   If it returns `404` or `401`, the endpoint is NOT accessible via LLAT.
3. **Write unit tests** (mocked) in `tests/unit/` — minimum 80% coverage for new code
4. **Add a smoke test** in `tests/smoke/` for basic functional verification
5. **Add an integration test** in `tests/integration/` for real HA validation

### The `get_automation_traces` Incident (v1.1.0)

- Tool was written assuming `/api/trace/context/` was a public REST endpoint
- All 6 unit tests used `patch("make_ha_request")` returning mocked `success: true`
- No curl verification against real HA
- Tool never worked in production; was removed in v1.1.1

**Lesson:** Mock-based unit tests are insufficient for API tools. Always verify with curl.

## HA API Authentication

### Critical: LLAT vs Frontend Auth

Home Assistant has **two separate authentication scopes** for its REST API:

| Auth Method | Scope | Use Case |
|-------------|-------|----------|
| `Authorization: Bearer <LLAT>` | Public REST API endpoints | Entity states, services, config check, templates, history, logbook |
| Frontend session cookie | Internal/frontend endpoints | Trace context, some config flows, UI-only endpoints |

### The LLAT vs Frontend Trap

**Long-Lived Access Tokens (LLATs)** do NOT have access to every endpoint that the frontend uses. Some endpoints (like `/api/trace/context/`) require a frontend session cookie and will return `404` or `401` when accessed with an LLAT.

### Before Implementing Any New HA API Tool

1. **Verify the endpoint in official docs** first:
   - [HA REST API docs](https://developers.home-assistant.io/docs/api/rest/)
   - If the endpoint is NOT listed there, it is **not a public REST API endpoint**

2. **Test the endpoint with curl BEFORE writing any code:**
   ```bash
   curl -s -H "Authorization: Bearer $HA_TOKEN" "http://HA_IP:8123/the/endpoint"
   ```
   If it returns `404` or `401`, the endpoint is not accessible via LLAT.

3. **Never assume** an endpoint exists only because you saw it in:
   - WebSocket API docs (different transport)
   - Frontend network tab (uses cookie auth)
   - Other Home Assistant API wrappers (may use different auth)

### Preventing Recurrence

After implementing any new HA API tool:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paulomac1000/ha-mcp-readonly](https://github.com/paulomac1000/ha-mcp-readonly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
