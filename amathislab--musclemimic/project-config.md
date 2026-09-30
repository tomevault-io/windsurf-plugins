---
trigger: always_on
description: This file applies to the entire repository. Add a nested `AGENTS.md` only when a
---

# MuscleMimic Development Guidelines

This file applies to the entire repository. Add a nested `AGENTS.md` only when a
subsystem needs stricter or more specific guidance.

## Project priorities

MuscleMimic is a JAX-based research codebase for muscle-actuated motion
imitation. Preserve these properties when changing it:

1. Correct physical and numerical behavior.
2. Reproducible training, evaluation, and checkpoint resume.
3. JAX/MJX/Warp performance without obscuring correctness.
4. Clear module ownership and stable public interfaces.
5. Small, reviewable changes with focused tests.

Do not trade correctness or reproducibility for a cleaner-looking abstraction.
When a refactor changes results, configuration semantics, compilation behavior,
or checkpoint compatibility, treat it as a behavior change and document it.

## Repository architecture

Use this target dependency direction:

```text
fullbody/ and bimanual/ entry points, scripts, analyses, examples
                              |
                              v
              runner, evaluation, and visualization
                              |
                              v
        algorithms, rl_core, environments, and musclemimic/core
                              |
                              v
       loco_mujoco simulation, trajectory, and data primitives
```

Responsibilities:

- `loco_mujoco/`: generic simulation, control, observation, trajectory, dataset,
  terrain, and retargeting primitives. It should not depend on application-level
  `musclemimic` training or runner code.
- `musclemimic/core/`: MuscleMimic-specific MJX state, goals, rewards, terminal
  handlers, and wrappers.
- `musclemimic/environments/`: environment and robot composition. Do not put
  training loops, logging backends, or CLI behavior here.
- `musclemimic/algorithms/` and `musclemimic/rl_core/`: learning algorithms,
  networks, optimizers, and rollout data structures. Keep environment-specific
  policy out of reusable algorithm code where practical.
- `musclemimic/runner/` and `musclemimic/evaluation/`: host-side orchestration,
  lifecycle, logging, checkpoint coordination, and evaluation.
- `musclemimic/viewer/` and `musclemimic/web_viewer/`: presentation and
  interactive tooling. Core simulation and algorithm modules must not depend on
  viewers.
- `fullbody/` and `bimanual/`: thin Hydra entry points and experiment
  configuration, not alternate implementations of shared behavior.
- `tests/`: behavior and regression coverage. Mirror source boundaries where it
  makes tests easier to find.

Some imports currently run against this direction. Treat them as existing
technical debt, not precedent. Do not introduce a new reverse dependency to
complete an unrelated task. If a boundary must be crossed, expose a small public
interface or move the genuinely shared concept to a neutral lower layer.

## Module and API design

- Depend on public interfaces. Do not import a leading-underscore symbol from
  another module.
- Keep `__init__.py` exports intentional and small. Avoid new wildcard imports
  and compatibility re-export chains.
- Before changing a public function, class, config key, registry name, or
  serialized field, search its uses in source, configs, tests, scripts, and
  examples.
- Preserve public signatures and configuration/checkpoint compatibility by
  default. If a break is necessary, provide migration guidance and, when
  practical, a deprecation path.
- Prefer cohesive modules over generic dumping grounds such as `utils.py`.
  Separate orchestration, domain computation, I/O, and presentation when they
  evolve independently.
- Extract one boundary at a time. First characterize existing behavior with a
  test; then move code without mixing in algorithmic changes.
- Avoid new dependencies unless the standard library or an existing dependency
  cannot reasonably solve the problem.
- Comments should explain intent, constraints, units, or non-obvious edge cases,
  not restate the code.
- Add type annotations to new public interfaces. Put types in signatures rather
  than repeating them in docstrings.
- Document units and array shape conventions for physical quantities at public
  boundaries, for example `[m]`, `[rad]`, `[N]`, and
  `[num_envs, num_joints]`.

## Documentation writing

All documentation uses concise, factual, analytical prose and active voice.
State claims directly. Replace rhetorical negation, contrastive pairings,
subjective qualifiers, and explanatory padding with functional descriptions.
Use no em dash or en dash. Reserve the ASCII hyphen for compound words,
hyphenation, literal commands, configuration keys, and mathematical syntax.
Rewrite clause breaks with a period, semicolon, colon, comma, or parentheses.
Keep formatting minimal. Use prose and code blocks when they carry technical
meaning. Omit tables, icons, emoji, decorative separators, marketing-style
headings, and visual padding.

## Python project quality and code clarity

- Goal: adopt project-grade quality norms common in high-volume open-source Python ML
  libraries and reproducible research codebases.

  Baseline references we align with:
  - PEP 8 (style, naming, imports, line breaks, comments, and code layout).
  - PEP 257 (docstring conventions).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amathislab/musclemimic](https://github.com/amathislab/musclemimic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
