---
trigger: always_on
description: This repository develops OCDraw and IFCCAD in parallel. OCDraw is an open,
---

# AGENTS.md

## Project context

This repository develops OCDraw and IFCCAD in parallel. OCDraw is an open,
application-independent information model and exchange format for standalone CAD
drawings; IFCCAD is an independent experimental CAD drawing profile using IFCX.
IFCCAD was formerly called IFCX-CAD; the repository name matches this track.
Project-owned API, folder, schema and crate names use IFCCAD; underlying IFCX
syntax and file extensions retain their IFCX names. Active code and schemas no longer require an
IFCX package or IFCDR/IFCPR resources. Initial OCDraw 0.1.0 remains provisional.

The active IFCCAD development track has its independent model under
`src/ifccad`, conversion under `crates/ifccad-convert`, and a separate browser
inspection/conversion route alongside OCDraw.
Keep its model, schemas, validation and conversion routes clearly separate from
standalone OCDraw. IFCX must not become a requirement for opening OCDraw.
Retiring the old package architecture does not retire this independent experiment.

Start with:

- `README.md` for the project purpose, drawing architecture, current
  capabilities, roadmap summary, and repository layout;
- `ROADMAP.md` for authoritative development sequencing, milestone status,
  dependencies, and exit criteria;
- `docs/vision.md` for long-term use cases, design principles, and success
  criteria;
- `crates/ocdraw-convert/README.md` for conversion terminology and the boundary
  between OCDraw and opencadcodec `CadDocument`;
- `crates/ocdraw-convert/docs/FROM-CAD-COVERAGE.md` for the pinned opencadcodec export
  coverage and loss-classification contract.

For measurement or encoding work, also read
`docs/benchmarks/ocdraw-size-exchange-v1.md` for the standalone controlled experiment, its limits,
and reproduction instructions. Import changes must also consult
`crates/ocdraw-convert/docs/TO-CAD-COVERAGE.md`.

For performance work, also read `docs/benchmarks/placement-preparation-v1.md`
for the current preparation measurements and practice-file inventory. Prefer
optimizations justified by measured representative workloads. Keep synthetic
stress cases for correctness and numerical boundaries; their cost alone does
not establish a practical priority. Further oblique-placement tuning and shared
placement preparation remain deferred until practice data and profiling justify them.

Use the following source-of-truth order:

- `schemas/` and `conformance/` define the language-neutral format contract;
- Rust source and tests define the current Rust API and implementation behavior;
- relevant documents under `docs/superpowers/specs/` explain approved design
  decisions and rationale, but are not a substitute for current code, schemas,
  or tests.

`ROADMAP.md` is the source of truth for development sequencing and milestone
status. The roadmap order expresses architectural dependencies even when
milestones overlap. Do not couple later work to unresolved earlier boundaries.
When proposed work crosses milestones, identify the dependency and trade-off
explicitly.

Dated specs and plans record the scope and status of their implementation
slice. Their historical "remaining work" statements do not override the current
roadmap or reopen completed milestones.

Open GitHub issues provide roadmap and future-design context, but are not
normative requirements. Before designing changes that affect format
architecture, schema evolution, physical encoding, preservation, or conversion
coverage, review the relevant open issues and identify overlaps or conflicts.
Do not expand the current task merely because a related issue exists.

## Local Superpowers workflow documents

`docs/superpowers/` stays local and outside Git. Its relevant designs and plans
are working documents for the Superpowers workflow: consult and maintain them
during the agent task to retain agreed decisions, execution steps and progress.
They are not disposable merely because they are untracked.

Record lasting contracts, architecture decisions and development conventions in
the regular repository documentation as well. After a task is complete, local
documents may be cleaned up once they contain no unique information needed for
follow-up work. Retain designs and plans still needed by active or upcoming work.

## Architecture boundaries

- The core `ocdraw` crate owns the drawing model, shared validation, typed
  construction, encoding, and standalone file storage.
- The core crate must remain independent of opencadcodec and other CAD runtimes.
- The `ocdraw-convert` companion crate owns conversion between validated OCDraw
  drawings and opencadcodec `CadDocument`.
- Keep conversion, logical package construction, physical encoding, and
  filesystem storage as separate responsibilities.
- Add new entity semantics to the language-neutral logical contract and shared
  semantic validation; keep physical field/range rules in the codec mapping.
  Exercise reader and writer backings through the shared logical access rather
  than duplicating semantic rules in each direction.
- Do not silently approximate or discard source semantics. Represent them,
  diagnose the loss, preserve them through an approved preservation mechanism,
  or reject structurally inconsistent input.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenAEC-Foundation/ifccad](https://github.com/OpenAEC-Foundation/ifccad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
