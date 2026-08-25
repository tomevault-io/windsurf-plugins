---
trigger: always_on
description: This file defines repository-level development rules for people and AI agents working in the standalone MetaSim repo.
---

# AGENTS.md

This file defines repository-level development rules for people and AI agents working in the standalone MetaSim repo.

The goals are:
- keep MetaSim core simple and easy to read
- prefer local consistency over clever abstractions
- follow existing project infrastructure instead of inventing parallel workflows
- keep downstream RoboVerse/content-pack concerns out of MetaSim unless the change is explicitly about integration

For testing infrastructure details, also refer to:
- [docs/source/developer_guide/autotest.md](./docs/source/developer_guide/autotest.md)
- [ENVIRONMENTS.md](./ENVIRONMENTS.md)
- [README.md](./README.md)

## General Development Rules

### Keep Changes Small And Local

- Prefer modifying existing files over adding new files.
- Do not introduce a new helper/module/doc file when an existing one already owns that responsibility.
- Do not add architectural layers or abstractions unless the current code clearly needs them.
- If a change can be expressed as a small local fix, prefer that over a framework-style refactor.

### Preserve Existing User-Facing Contracts

- Do not silently change public semantics when the repo already has established behavior.
- When broadening a base contract, make sure concrete subclasses actually satisfy it.
- If a behavior is only compatibility support, document it as such instead of treating it as a primary contract.

### Keep MetaSim Core Scoped

- MetaSim owns the core simulator abstractions, scenario configuration types, task registry, package discovery utilities, built-in example assets, and simulator backends.
- Do not add RoboVerse task, robot, scene, or ground pack logic directly into MetaSim core unless the change is explicitly about core compatibility with downstream packages.
- Prefer existing package discovery mechanisms for downstream content: entry points, `metasim.toml`, `[tool.metasim.packages]`, and `METASIM_*_PACKAGES`.
- Avoid hard dependencies on downstream content packs in core tests and docs unless the file is specifically documenting compatibility behavior.

### Be Explicit About Scope

- Only test simulators that are related to or influenced by the current change.
- If the correct simulator env mapping is not known from repo context or user instructions, ask before running simulator-backed tests.
- Do not claim "all tests pass" unless the exact requested test commands were actually run in the correct envs.

### Prefer Fail-Fast Over Silent Degradation

- For unsupported backends or unsupported code paths, prefer an explicit error over silent success.
- Do not turn an explicit unsupported-operation failure into a silent no-op unless there is a strong reason.

## Testing Rules For `metasim/test`

### Source Of Truth

The authoritative test workflow is the MetaSim autotest guide:
- [docs/source/developer_guide/autotest.md](./docs/source/developer_guide/autotest.md)

When there is any doubt about pytest structure, markers, suite registration, handler reuse, or backend-specific commands, follow that guide.

### Test Discovery And File Format

For files under `metasim/test`:

- Test file names should start with `test_` or end with `_test.py`.
- Test function names should start with `test_`.
- Helper functions should not start with `test_`.
- Test directories that participate in the shared suite/registration system must contain `__init__.py`.
- If a suite uses the shared `handler` fixture, its package or file prefix must be registered through `register_shared_suite(...)` in the relevant `conftest.py`.

### Markers And Fixtures

Use the existing marker model described in the autotest guide and `metasim/test/conftest.py`:

- `@pytest.mark.mujoco`
- `@pytest.mark.mjx`
- `@pytest.mark.isaacgym`
- `@pytest.mark.isaacsim`
- `@pytest.mark.newton`
- `@pytest.mark.sapien3`
- `@pytest.mark.blender`
- `@pytest.mark.sim(...)`
- `@pytest.mark.general`

Rules:

- `@pytest.mark.general` tests should not request the `handler` fixture.
- Simulator-backed tests should use the repo's shared fixture/registration system instead of custom one-off setup.
- Prefer extending an existing relevant test file under `metasim/test` before creating a new one.

### Backend-Specific Autotest Commands

When running tests for `metasim/test`, use these commands in the corresponding conda envs from [ENVIRONMENTS.md](./ENVIRONMENTS.md):

```bash
# MuJoCo only
pytest metasim/test/ -k mujoco

# MJX only
pytest metasim/test/ -k mjx

# IsaacGym (use entry script for proper import order)
python metasim/test/isaacgym_entry.py metasim/test/ -k isaacgym

# IsaacSim only
pytest metasim/test/ -k isaacsim

# Newton only
pytest metasim/test/ -k newton

# Blender only (requires `bpy`)
pytest metasim/test/ -k blender

# General tests (no simulator)
pytest metasim/test/ -k general
```

For `sapien3` tests, first confirm the local SAPIEN environment mapping in [ENVIRONMENTS.md](./ENVIRONMENTS.md) or with the user, then run the relevant `pytest metasim/test/ -k sapien3` selection in that environment.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RoboVerseOrg/MetaSim](https://github.com/RoboVerseOrg/MetaSim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
