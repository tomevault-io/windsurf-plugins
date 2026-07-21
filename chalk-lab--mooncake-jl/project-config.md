---
trigger: always_on
description: Mooncake.jl is a Julia-first automatic differentiation package focused on:
---

# AGENTS.md

## Purpose

Mooncake.jl is a Julia-first automatic differentiation package focused on:

- broad coverage of real Julia behaviour, especially mutation, dynamic control flow, foreign calls, intrinsics, arrays, structs, tasks, closures, and package-extension code
- correctness and testability before aggressive optimisation, verified empirically through wide test coverage and tangent-type design
- composability: rules should compose predictably across primitives, custom tangents, nested AD, and mixed-mode AD
- representation discipline: tangent and cotangent types should be canonical enough that invariants are easy to state, test, and preserve
- strong diagnostics: malformed rules, tangent mismatches, world-age/compiler issues, and mutation mistakes should be easy to surface and debug
- clear validity boundaries: unsupported cases should fail loudly and locally, not silently produce wrong derivatives
- numerical robustness, including removable-singularity cases that would otherwise produce NaNs/Infs
- performance via hand-written low-level `rrule!!` / `frule!!`, strict tangent and cotangent types, and cached prepare/run APIs

The overall target is: correct by construction where possible, aggressively testable where not, and explicit about every place where semantics depend on a rule.

## Repository Layout

- `src/`: main package code
- `src/interpreter/`: IR and interpreter machinery for forward and reverse mode
- `src/rules/`: primitive- and domain-specific differentiation rules
- `ext/`: package extensions
- `test/`: core test suite
- `test/ext/`: extension tests in separate environments
- `test/integration_testing/`: broader integration suites in separate environments
- `docs/src/`: user, conceptual, utility, and developer documentation

## Working Conventions

- Keep changes aligned with the existing source/test layout: tests for `src/.../foo.jl` usually live at `test/.../foo.jl`.
- Put shared test setup in `test/front_matter.jl`; test-group dispatch lives in `test/runtests.jl`.
- For complex rules, especially array-heavy rules, prefer canonicalising inputs at the rule boundary with utilities such as `arrayify` rather than proliferating specialised methods.
- Mooncake provides helpers for importing rules from ChainRules via `@from_rrule` / `@from_chainrules`, but use them conservatively. In practice, restrict to scalar and array-like cases whose element types are `IEEEFloat` or `Complex` numbers, for which tangent conversions are well-defined and round-trip correctly.
- World-age issues can arise when generated functions call back into Julia dispatch. `tangent_type` and `build_fdata` are generated functions; all sub-function calls must be in the returned expression (runtime), not in the generator body (generation time). If you add or modify either function, verify this.
- Avoid modifying `src/interpreter/` unless the task explicitly targets it. `Mooncake.primal_ir`, `Mooncake.dual_ir`, `Mooncake.fwd_ir`, and `Mooncake.rvs_ir` (see `docs/src/developer_documentation/developer_tools.md`) are available for inspection, but do not write rules or code that depends on their output — they are not semver-stable.
- Prefer writing rules at the lowest practical level, often around foreign-call boundaries (see `src/rules/blas.jl`), to reduce the total number of rules that need to be maintained.
- Implement both `frule!!` and `rrule!!` for new primitives where possible; rules that cover only one mode limit composability.
- Every custom rule must be accompanied by an `@is_primitive` declaration; without it, the AD will not dispatch to the rule.
- Use `@zero_derivative` for rules with a zero derivative rather than writing a manual rule. Check `src/rules/` for other convenience macros before writing a rule from scratch.
- When choosing a tangent type: use `NoTangent` for non-differentiable types (e.g. integers, booleans, symbols); use `ZeroTangent` when the type is differentiable, but the derivative is structurally zero in a given rule.
- Prefer the narrowest rule signature that covers the intended cases; overly broad signatures can silently shadow more specific rules or cause method ambiguity errors.
- Only forward-over-reverse nested AD is tested. Do not assume rules compose correctly under reverse-over-reverse or other higher-order combinations unless explicitly verified.
- Perturbation confusion / CUDA higher-order limit: GPU kernels are foreign calls with no Julia IR, so the CUDA extension differentiates elementwise/reduction ops (`sum(f, x)`, broadcasts) by running the mapped function on `Nfwd.NDual` forward-mode numbers inside the kernel. `NDual` is untagged and cannot nest (`NDual{NDual}` raises an error), so HVP/Hessian (forward-over-reverse) through these rules would collide the two perturbation levels — perturbation confusion. They error loudly instead (never silently wrong); HVP/Hessian for non-elementwise ops (`sum(x)`, `dot`, BLAS) works. Fix: a nested second-order dual (`HyperDual`).
- Prefer clear Julia error messages, especially around malformed rules, unsupported cases, and rule-construction failures.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chalk-lab/Mooncake.jl](https://github.com/chalk-lab/Mooncake.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
