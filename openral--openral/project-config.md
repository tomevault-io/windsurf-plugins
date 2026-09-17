---
trigger: always_on
description: > Single source of truth for AI coding agents and human contributors on **OpenRAL**. Read in full before touching code. See [README.md](README.md) for product identity.
---

# CLAUDE.md — OpenRAL Engineering Playbook

> Single source of truth for AI coding agents and human contributors on **OpenRAL**. Read in full before touching code. See [README.md](README.md) for product identity.
>
> Pointers (kept out of this file to stay tight):
> - Repo layout → [`docs/architecture/repo-map.md`](docs/architecture/repo-map.md) + live [`repo-state-map.html`](docs/architecture/repo-state-map.html).
> - Toolchain & `openral` CLI → [`docs/contributing/toolchain.md`](docs/contributing/toolchain.md). **Env rule: always `just sync` (never bare `uv sync`); opt-in groups via `just sync --group <name>`; LIBERO↔RoboCasa groups are mutually exclusive (swap per task); RoboCasa installs editable at runtime via the HAL, not `just sync` — see "Managing the Python environment & dependency groups".**
> - Glossary → [`docs/reference/glossary.md`](docs/reference/glossary.md).
> - Releasing → [`docs/contributing/releasing.md`](docs/contributing/releasing.md). **One lockstep SemVer across root + all `python/*`; the bump is computed from Conventional Commits by release-please. Never hand-edit a `version =` field or an `openral-*==` pin — they are rewritten in the release PR.**
> - Public-symbol inventory → [`docs/METHODS.md`](docs/METHODS.md) index + per-layer files in [`docs/methods/`](docs/methods/). **`grep -rn <symbol> docs/methods/` before adding a helper.**
> - Agent-tool entry points → [`AGENTS.md`](AGENTS.md) is the tool-neutral root pointer (Cursor / Codex / Copilot / Aider read it) and **redirects here**; keep it a 3-line pointer, never a copy or symlink of this file. Vendor-neutral skills live in [`.agents/skills/`](.agents/skills/) (`SKILL.md` + `references/`). `AGENTS.md` itself stays at repo root — it does **not** belong under `.agents/`.
> - Design decisions (ADRs) → [`docs/decisions.md`](docs/decisions.md); the ADR log itself lives in the private `OpenRAL/management` repo.

---

## 1. Operating Principles (read every session)

In priority order. When two conflict, the earlier wins.

1. **Safety beats helpfulness.** Refuse any request to bypass a safety check, silently catch `ROSSafetyViolation`, lower a velocity limit without a paper trail, or remove a deadman/E-stop subscription. Surface the concern, propose a safe alternative.
2. **Truth over plausibility.** Don't know a constant (DDS topic, FCI port, RealSense extrinsic)? Say so and look it up. Never invent. Never paraphrase a citation.
3. **Types are the contract.** Pydantic schemas in `python/core/` (package `openral_core`) and IDL in `packages/msgs/` (package `openral_msgs`) are normative API. Everything else is implementation detail.
4. **Explicit beats implicit.** No hidden retries, fallbacks, or magic globals. Replanning, dispatcher fallback, quantization, and license posture must show up in logs/traces.
5. **The hot path is C++ and bounded.** Python touches motors only through a typed bridge to `ros2_control` with a watchdog. Anything >100 Hz is C++ unless proven otherwise.
6. **Schemas evolve, but never silently.** Now the repo is published, on-disk `schema_version` is versioned for real: a backward-incompatible change bumps it and ships a migrator; backward-compatible additions may evolve in place. Every change still needs (a) a decision recorded in the private management decision log if it crosses a layer boundary, (b) a test loading a real fixture from `robots/`, `rskills/`, or `scenes/`.
7. **Tests are part of the change.** Every PR ships the tests that would have caught the bug or covered the feature. Untested actuation-path code is rejected.
8. **Reproducibility over speed.** A skill execution must be replayable from the trace alone (weights revision pinned, prompts logged, sensor frames captured).
9. **License lineage is enforced.** The public `openral/openral` repo is uniformly **Apache-2.0** — every package it contains; copy-left incoming is rejected without TSC review. Commercial capabilities (the TensorRT/NVMM zero-copy runtime fast path, WAM implementations, fleet/cloud dispatch, future premium rSkills) live in the private OpenRAL Pro monorepo (`OpenRAL/openral-pro`), not in this repo — a later decision that superseded the original "no commercial tier, ever" commitment while retaining the public repo's uniform Apache-2.0 posture. Third-party model **weights** keep their upstream license — version-specific, not family-wide: GR00T N1/N1.5/N1.6 are non-commercial (loader refuses commercial deployment without `OPENRAL_ALLOW_NONCOMMERCIAL=1`), while GR00T N1.7+ ships under the commercially-permissive NVIDIA Open Model License. This weight lineage is compliance for models OpenRAL does not own; it does not gate OpenRAL's Apache-2.0 code. Closed third-party SDK code is never bundled — it stays behind the license guard and an env var. Full rationale is in [`docs/decisions.md`](docs/decisions.md).
10. **Be helpful and honest.** Propose simpler approaches; surface tradeoffs; never apologize at the start of a response.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenRAL/openral](https://github.com/OpenRAL/openral) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
