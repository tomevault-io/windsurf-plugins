---
trigger: always_on
description: This is a Django-based cloud orchestration platform. When working on this codebase:
---

# Waldur Development Guide for Claude

## Quick Start

This is a Django-based cloud orchestration platform. When working on this codebase:

1. **Study existing patterns first** - Find 3 similar implementations before coding
2. **Follow TDD** - Write tests first, then implement
3. **Use established patterns** - Don't reinvent what already exists
4. **Stop after 3 attempts** - If stuck, document and try different approach

## Critical Rules

**NEVER**:

- Use `--no-verify` to bypass commit hooks
- Disable tests instead of fixing them
- Commit code that doesn't compile
- Make assumptions - verify with existing code
- Import modules inside functions - all imports must be at the top of the file
  (narrow exception: lazy imports of heavy optional backend SDKs — see
  "Lazy imports for heavy optional backends" below)

**ALWAYS**:

- Use `permission_factory` for ViewSet permissions
- Use `SlugRelatedField(slug_field="uuid")` for relationships
- Test actual system behavior, not assumptions
- Run tests and linters before committing

## Key Waldur Patterns

### Lazy imports for heavy optional backends

The default rule is imports-at-top-of-file. The **only** sanctioned exception is
the heavy SDK of an optional provider backend that most deployments never use.
Such SDKs (e.g. `azure-mgmt-*` ≈ 37 MB, `apache-libcloud`, large cloud clients)
get imported in **every** process at Django startup if referenced at module top —
because the provider's `apps.py` `ready()` registers the backend, which imports
`backend.py` → `client.py`. That permanently inflates the resident memory of API
and Celery pods for a feature they don't exercise.

When (and only when) **all** of these hold, defer the SDK import:

- the dependency is large and belongs to an **optional** provider backend;
- it is used only inside `client.py` / `backend.py` methods (never in
  `models.py` / `serializers.py` / `views.py` / `urls.py` / `apps.py`);
- deployments that don't use the provider should not pay for it.

How to defer (keep it disciplined — this is not licence for ad-hoc local imports):

- Put the `from azure... import X` **inside the method** that uses it.
- For symbols used in many `except` clauses, add one module-level helper that
  imports lazily, e.g. `def _azure_exceptions(): from azure.core.exceptions
  import AzureError, HttpResponseError; return AzureError, HttpResponseError`,
  then `except _azure_exceptions() as exc:`.
- Add `from __future__ import annotations` so SDK types used only in annotations
  stay lazy; put those imports under `if TYPE_CHECKING:` for the type checker.
- Leave a comment at the top of the file pointing back to this section.

Reference implementation: `src/waldur_azure/client.py` and `backend.py`. Measure
the before/after with `scripts/measure_startup_memory.py` (the per-component
heatmap shows whether the SDK still loads at startup).

### Permissions

```python
# Use permission_factory, not manual checks
list_permissions = [permission_factory(PermissionEnum.VIEW_RESOURCE)]
```

**Adding new permissions:**
1. Add to `PermissionEnum` in `src/waldur_core/permissions/enums.py`
2. Assign to roles in `docker/rootfs/etc/waldur/permissions.yaml` (NOT via data migrations)
3. The `import_roles` management command loads permissions.yaml on deployment

See `docs/guides/waldur-permissions.md` for details.

### Serializers

```python
# Use SlugRelatedField for UUIDs
project = serializers.SlugRelatedField(slug_field="uuid", queryset=Project.objects.all())

# CRITICAL: Nullable FKs MUST use allow_null=True on SlugRelatedField
# Without it, the OpenAPI spec won't mark the field as nullable,
# and auto-generated SDK clients will crash on null values (e.g. UUID(None))
created_by = serializers.SlugRelatedField(slug_field="uuid", read_only=True, allow_null=True)
```

### Testing

```python
# Use established fixtures
fixture = fixtures.ProjectFixture()
# Use real roles
role = CustomerRole.SUPPORT  # Not MANAGER (doesn't exist)
```

### Demo Presets

Demo presets are JSON files in `src/waldur_mastermind/marketplace/demo_presets/presets/` that define complete marketplace ecosystems for testing and demos.

**UUID Format Rules** (CRITICAL):

- UUIDs must be **exactly 32 hexadecimal characters** (0-9, a-f only)
- **NO hyphens** - use continuous string format
- **NO letters g-z** - these are not valid hex characters
- All `*_uuid` reference fields must match the referenced entity's UUID exactly

```python
# CORRECT UUID format
"uuid": "afc00000000000000000000000000001"  # 32 hex chars

# WRONG - contains non-hex letters
"uuid": "afk00000000000000000000000000001"  # 'k' is not hex
"uuid": "af3plan0000000000000000000000001"  # 'p', 'l', 'n' are not hex

# WRONG - wrong length
"uuid": "afc0000000000000000000000000001"   # 31 chars (missing one)
```

**Reference Consistency**: When referencing entities via `*_uuid` fields (e.g., `customer_uuid`, `offering_uuid`, `plan_uuid`), ensure the UUID exactly matches the target entity's `uuid` field.

**Commands**:

```bash
# List available presets
waldur demo_presets list

# Load a preset (destructive - clears existing data)
DJANGO_SETTINGS_MODULE=waldur_core.server.test_settings_local waldur demo_presets load <name> -y

# Dry run (preview without changes)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [waldur/waldur-mastermind](https://github.com/waldur/waldur-mastermind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
