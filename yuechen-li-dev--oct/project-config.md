---
trigger: always_on
description: Before editing, read the smallest relevant slice of the repo truth:
---

# AGENTS.md

## Quick field manual

Before editing, read the smallest relevant slice of the repo truth:

- `README.md` for user-facing workflow and release positioning.
- First-party `manifest.oct` files should include ordered `Authors: String[]` and ISO `Date: String` metadata; preserve explicit author policy when updating package manifests.
- `Language/reference/...` for Oct syntax, style, and supported language features.
- Relevant `Language/...` fixtures; they are examples **and** semantic contracts.
- Relevant `docs/internal/...` notes for release, package-manager, wrapper, or compiled-backend work.

For EVT-2 Z-Image work, read `docs/EVT2_LOCAL_PAYLOADS.md` before reporting
missing local payloads. Unset environment variables are ordinary setup, not an
architectural blocker.

Testing guidance:

- Language behavior belongs in `.octest` / `.octfail` corpus tests under `Language/`; prefer adding or updating those contracts over embedding Oct programs in Go tests.
- If you change Go code, run targeted Go tests plus the relevant Oct language/library corpus lane.
- If you only change docs, fixtures, or Oct examples, a full `go test ./...` is usually unnecessary; run the relevant `oct test` corpus/examples instead.
- Do not force tests to interpreted mode to hide compiled failures. Do not weaken no-fallback or missing-sidecar assertions. If compiled support is absent, document that explicitly.

Octxiliary/wrapper workflow:

- Build sidecars explicitly with `go run ./tools/build_sidecars --out dist/sidecars`.
- Run slow wrapper lanes explicitly with `OCT_WRAPPER_PATH` and `OCT_SLOW_TESTS=1` (or the legacy `OCT_RUN_SLOW_TESTS=1`).
- Normal feature PRs should avoid broad slow wrapper lanes unless wrapper/octxiliary code changed or the lane was requested.
- Keep generated and scratch artifacts out of commits: do not commit local `dist/`, temporary outputs, `.oct/` caches, or test artifacts.

Release hygiene:

- Prefer small surgical commits.
- If touching release/tagging, follow `docs/internal/release_readiness_0_1.md`.
- Do not create or push `v0.1.0` unless the release checklist is green and a human explicitly approves tagging.


## Language Reference Authority Rule (Oct Code)

* The `Language/reference` directory is the **single source of truth** for:

  * syntax
  * style conventions
  * supported language features

* When writing or modifying Oct code, **always follow `Language/reference`**, even if:

  * existing experiments
  * older library code
  * prior generated code

  use different patterns.

* Treat older code as potentially outdated. Do **not** cargo-cult:

  * deprecated syntax
  * superseded patterns (e.g., while-loops used as for-loops)
  * pre-fix workarounds

* If you encounter inconsistencies between:

  * `Language/reference`
  * existing code
  * experiments
  * tests

  you must:

  1. **Follow `Language/reference`**
  2. **Explicitly surface the inconsistency** in your report or summary

* If a feature appears in code but is **not documented** in `Language/reference`, or vice versa:

  * treat this as a documentation gap
  * call it out explicitly

* Do not silently resolve inconsistencies.
  Always make them visible so they can be corrected intentionally.

## Primer Rule (Native Code)

Read `primer/` before writing or editing native code (C, C++, Vulkan, etc.).

The files in `primer/` are the authoritative coding rules for native code in this repository.
Do not write native code that conflicts with them.
Do not substitute your own preferred style for the primer rules.
Do not skip testing. If Vulkan is not availble inside sandbox, attempt to download Vulkan from apt. Surface issue explicitly if testing is truly impossible. 

If instructions and primers appear to disagree, surface the conflict explicitly.

## Convergence rule

Every substantial task must end in exactly one of three states:

1. **Success**  
The intended capability works in the real path and the real motivating case materially improves.
2. **Meaningful progression**  
The capability is not complete, but one genuine blocker is removed and the next blocker is isolated with evidence.
3. **Honest stop**  
Further work would require overbroad scope expansion, excessive debt, brittle patching, or tangled logic. Stop and report the reason with concrete evidence.

Do not continue producing patches once the work stops converging.

Do not confuse activity with progress.
A failed attempt is only acceptable if it leaves behind a narrower problem, stronger evidence, or a justified stop.

Any partial work must leave the codebase in a cleaner, more legible, and more diagnosable state than before.

## Judgment Rule

When a compiler/tooling decision has:
- one selected answer,
- a bounded set of valid candidate choices,
- multiple competing input signals,
- no single signal that should always dominate,
- and a need for deterministic/debuggable behavior,

prefer internal/judgment over fragile nested if/else ladders.

Examples:
- formatter layout choice:
  inline vs multiline vs leaveUnchanged

- diagnostic suggestion choice:
  unknown function vs missing import vs wrong namespace vs typo suggestion

- import/path recovery:
  candidate package roots or fixture paths

- artifact/report presentation:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yuechen-li-dev/oct](https://github.com/yuechen-li-dev/oct) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
