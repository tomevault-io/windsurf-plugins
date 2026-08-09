---
trigger: always_on
description: - **Breaking changes require a deprecation first.** Do not remove or rename public API symbols without deprecating them in a prior release.
---

# IsaacLab Guidelines

## Breaking API changes

- **Breaking changes require a deprecation first.** Do not remove or rename public API symbols without deprecating them in a prior release.

## API design rules (naming + structure)

- **Group by common prefix for discoverability (autocomplete).**
  - **Classes**: group by domain concept — `ActuatorNetLSTM`, `ActuatorNetMLP` (not `LSTMActuatorNet`, `MLPActuatorNet`).
  - **Methods**: group by noun before modifier — `set_joint_position_target()` (not `set_target_joint_position()`).
- **Method names are `snake_case`.**
- **CLI arguments are `snake_case`.**
- **Prefer nested classes when self-contained.**
  - If a helper type or an enum is only meaningful inside one parent class and doesn't need a public identity, define it as a nested class instead of creating a new top-level class/module.
- **Follow PEP 8 for Python code.**
- **Use modern Python type-hint syntax.**
  - Prefer PEP 604 unions: `x | y`, `x | None`. Do not use `typing.Union` or `typing.Optional`.
- **Use specific type hints for public interfaces.**
  - For torch tensors, annotate with `torch.Tensor`. For Warp arrays, annotate concrete dtypes (e.g., `wp.array(dtype=wp.vec3)`) rather than generic `object`.
  - Prefer consistent parameter names across base/override APIs (e.g., `xforms`, `scales`, `colors`, `materials`).
- **Use Google-style docstrings.**
  - Write clear, concise docstrings that explain what the function does, its parameters, and its return value.
  - Keep argument/return types in function annotations, not inline in docstrings.
  - In `Args:` entries, use `name: description` (not `name (Type): description`).
  - Use Sphinx cross-reference roles for symbol references (e.g. `:class:`, `:meth:`, `:attr:`, `:paramref:`), but keep targets as short as possible.
  - Within the same class/module, prefer short local references (e.g. `:meth:\`set_joint_position_target\``, `:attr:\`num_joints\``) over fully qualified paths.
  - If qualification is needed, prefer public API paths (e.g. `isaaclab.assets.Articulation`) and do not use internal `_src` or private module paths in Sphinx role targets.
- **State SI units for all physical quantities in docstrings.**
  - Use inline `[unit]` notation, e.g. `"""Particle positions [m], shape [particle_count, 3], float."""`.
  - For joint-type-dependent quantities use `[m or rad, depending on joint type]`.
  - For spatial vectors annotate both components, e.g. `[N, N·m]`.
  - For compound arrays list per-component units, e.g. `[0] k_mu [Pa], [1] k_lambda [Pa], ...`.
  - When a parameter's interpretation varies across solvers, document each solver's convention instead of a single unit.
  - Skip non-physical fields (indices, keys, counts, flags).
  - This rule applies to **public API docstrings only**, not test docstrings.
- **Keep the documentation up-to-date.**
  - When adding new files or symbols that are part of the public-facing API, make sure to keep the auto-generated documentation updated by running `./isaaclab.sh -d`.

## Dependencies

- **Avoid adding new required dependencies.** IsaacLab's core should remain lightweight and minimize external requirements.
- **Strongly prefer not adding new optional dependencies.** If additional functionality requires a new package, carefully consider whether the benefit justifies the added complexity and maintenance burden. When possible, implement functionality using existing dependencies, including Warp functions and kernels, NumPy, or the standard library.

## Tooling: prefer `./isaaclab.sh -p` for running, testing, and benchmarking

We use a wrapped python call within `./isaaclab.sh`.

- **Use `./isaaclab.sh -p -c` for inline Python**: When running one-off Python commands, use `./isaaclab.sh -p -c "..."` instead of `python3 -c "..."`.
- **Use `./isaaclab.sh -p`** to run standalone Python scripts without a `pyproject.toml` (e.g., in CI after switching to a branch with no project files).

### Run tests

```bash
# run all tests (extremely heavy, should be avoided).
./isaaclab.sh -t

# run a specific test file by name
./isaaclab.sh -p -m pytest PATH_TO_TEST

# run a specific example test
./isaaclab.sh -p -m pytest PATH_TO_TEST::METHOD
```

### Pre-commit (lint/format hooks)

**CRITICAL: Always run pre-commit hooks BEFORE committing and BEFORE pushing.**

Proper workflow:
1. Make your code changes
2. Run `./isaaclab.sh -f` to check ALL files
3. If pre-commit modifies any files (e.g., formatting), review the changes
4. Stage the modified files with `git add`
5. Run `./isaaclab.sh -f` again to ensure all checks pass
6. Only then create your commit with `git commit`
7. Verify pre-commit still passes before pushing — never push commits that haven't been checked

```bash
# Run pre-commit checks on all files
./isaaclab.sh -f
```

**Common mistakes to avoid:**
- Don't commit first and then run pre-commit (requires amending commits)
- Don't push before running pre-commit (pushes broken code to the remote)
- Do run pre-commit before committing and before pushing (clean workflow)

**When reviewing code** (e.g. via a code-reviewer agent), always run `./isaaclab.sh -f` as part of the review to catch formatting or lint issues early.

## Changelog

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [isaac-sim/IsaacLab](https://github.com/isaac-sim/IsaacLab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
