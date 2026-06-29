---
trigger: always_on
description: Agent guidance for NiPyAPI - Apache NiFi Python Client SDK.
---

# AGENTS.md

Agent guidance for NiPyAPI - Apache NiFi Python Client SDK.

**Which workflow applies?**
- Using nipyapi in another project → [Consuming nipyapi](#consuming-nipyapi)
- Working in the nipyapi repository → [Contributing to nipyapi](#contributing-to-nipyapi)

---

## Consuming nipyapi

For agents using nipyapi to perform NiFi operations.

### Install

```bash
uv pip install "nipyapi[cli]"    # Recommended: includes CLI
pip install nipyapi              # Alternative: library only
```

### Connect

```bash
# Option 1: Profile (recommended)
export NIPYAPI_PROFILE=my-profile
# Create profile in ~/.nipyapi/profiles.yml - see examples/profiles.yml

# Option 2: Environment variables
export NIFI_API_ENDPOINT=https://nifi.example.com/nifi-api
export NIFI_USERNAME=admin
export NIFI_PASSWORD=secret
```

### Documentation

| If you need to... | See |
|-------------------|-----|
| Install nipyapi | `README.rst` → "Quick Start" |
| Configure profiles | `examples/profiles.yml`, `docs/profiles.rst` |
| Authenticate (Basic, mTLS, LDAP, OIDC) | `docs/security.rst` |
| Migrate from 0.x | `docs/migration.rst` |
| Full API reference | https://nipyapi.readthedocs.io |
| CLI usage and options | `nipyapi --help`, `nipyapi <module> --help` |
| Function signatures and behavior | Read module docstrings directly (comprehensive) |

### Modules

Prefer in this order: `nipyapi.ci` → helper modules → low-level API.

For function discovery: `nipyapi <module> --help` (CLI) or `help(nipyapi.canvas)` (Python).

| If you need to... | Use |
|-------------------|-----|
| Deploy flows in CI/CD | `nipyapi.ci` |
| Manage processors, process groups, connections | `nipyapi.canvas` |
| Import/export versioned flows | `nipyapi.versioning` |
| Authenticate, manage users/policies | `nipyapi.security` |
| Work with parameter contexts | `nipyapi.parameters` |
| Switch connection profiles | `nipyapi.profiles` |
| Position components on canvas | `nipyapi.layout` |
| Get system/cluster info | `nipyapi.system` |
| Retrieve/filter/clear bulletins | `nipyapi.bulletins` |
| Manage NiFi extensions (NARs) | `nipyapi.extensions` |
| File operations, filtering, common utilities | `nipyapi.utils` |
| Access/modify endpoint configuration | `nipyapi.config` |
| Direct NiFi API access (when helpers insufficient) | `nipyapi.nifi.*` |
| Direct Registry API access (when helpers insufficient) | `nipyapi.registry.*` |

### CLI Examples

```bash
nipyapi ci ensure_registry --name my-registry --uri https://github.com/org/repo
nipyapi ci deploy_flow --registry_client my-registry --bucket flows --flow my-flow
nipyapi ci start_flow --pg_id <process-group-id>
nipyapi canvas list_all_process_groups
```

---

## Contributing to nipyapi

For agents modifying the nipyapi codebase.

### Project Context

This project is nearly 10 years old with a substantial user base. Approach changes with care:

- **Existing patterns may handle edge cases** you're not aware of - ask before "simplifying"
- **Some code predates modern Python** (originated in Python 2 era) - refactoring requires testing
- **Changes can have broad impact** - prefer surgical modifications over sweeping rewrites
- **When in doubt, discuss first** - the PLAN→ACT→REVIEW workflow exists for good reason

### Setup

```bash
git clone https://github.com/Chaffelson/nipyapi.git && cd nipyapi
make dev-install    # Uses uv if available, pip otherwise
make help           # See all available targets
```

**Important**:
- Always activate venv first: `source .venv/bin/activate` before running Python/make commands
- Prefer make commands: Check `make help` for available targets before using generic tooling (pip, pytest, sphinx, etc.)

### Intent Routing

| If you need to... | See |
|-------------------|-----|
| Set up dev environment | `docs/contributing.rst` → "Get Started!" |
| Understand code organization | `docs/contributing.rst` → "Generated vs Maintained Code" |
| Run tests | `docs/contributing.rst` → "Make Targets Quick Reference" |
| Add a new module | `docs/contributing.rst` → "Adding New Core Modules" |
| Submit a PR | `docs/contributing.rst` → "Pull Request Guidelines" |
| Understand profiles system | `docs/profiles.rst` |
| Migrate from 0.x | `docs/migration.rst` |

### Before Writing Code

**Discovery pattern** - check before implementing new helpers:

1. Read `nipyapi/__init__.py` for module intent mapping (comments describe each module's purpose)
2. Check `__all__` at top of relevant module (`utils.py`, `canvas.py`, etc.)
3. Grep function names that might serve your purpose
4. Read docstrings to understand intent and edge cases handled
5. For tests: check `tests/conftest.py` for fixtures, read an example test file first

See `docs/contributing.rst` → "Reuse Existing Code" for detailed guidance.

### Critical Rules

**Will cause failures if ignored:**

1. **Profile required for tests**: `make test NIPYAPI_PROFILE=<profile>` - never bare pytest
2. **Docker readiness**: Always `make wait-ready NIPYAPI_PROFILE=<profile>` before testing
3. **Certificates before containers**: Run `make down` before `make certs`
4. **Zsh bracket quoting**: Use `make dev-install` not bare `pip install -e .[dev]`

### Code Organization

**Maintained** (where to contribute):
- `nipyapi/*.py` - Core modules
- `tests/` - Test suite

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Chaffelson/nipyapi](https://github.com/Chaffelson/nipyapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
