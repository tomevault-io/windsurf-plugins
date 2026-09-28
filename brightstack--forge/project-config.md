---
trigger: always_on
description: Forge is an agent-invoked delivery skill with professional instructions,
---

# Forge

Forge is an agent-invoked delivery skill with professional instructions,
human-readable templates, deterministic local mechanics, and behavioral evals.
Its CLI also serves a selected artifact directory through a foreground localhost
preview. Browser interaction and screenshots remain host capabilities.
It is not an application server, semantic command router, workflow daemon, or
agent-process launcher.

## Development and runtime

When developing or running Forge inside a host repository, that repository's
applicable `AGENTS.md` hierarchy and linked standards govern implementation. They
are not runtime dependencies of the portable skill.

Enter runtime behavior through [skills/forge/SKILL.md](skills/forge/SKILL.md) and
load only the relevant phase references, professional instructions, and templates.

Forge Review is the ordinary integrated review route. It selects applicable
dimensions from the [judge selection and rubrics](skills/forge/references/judges.md).
Use [workflow staffing](skills/forge/references/workflows.md#concrete-staffing)
to assign Reviewer and required Judges, and preserve their reports. The sibling
[forge-code-review](skills/forge-code-review/SKILL.md) skill is the portable
standalone leaf for an explicitly requested code-only review; it ends at its own
read-only Code Review verdict and never starts Forge Review or Acceptance.

## Runtime contract

Full delivery accounts for `Spec → Plan → Build → Acceptance → Ship`. Spec owns
intake. Build owns implementation, integration, simplification, internal review,
independent Review, and correction. Acceptance owns actual candidate proof. Each
phase can be invoked directly and stop. Explore, Review, Simplify, Finish, Spec apply, the
legacy natural wording Spec merge, and KB maintenance are direct operations
without implied full-delivery status.

Start with [Launch](skills/forge/references/workflows.md): Workflow is Project,
Issue, Bug, or Work; Depth is Quick or Full; Control is Guided or explicitly Auto.
Guided waits before dispatch, then at new Spec/consequential Plan gates. Auto
exercises its grant without inventing approval or overriding protected-Spec rules.
An Issue's complexity governs depth. A clear current request can supply ready
intent; PM fills missing product intent only. Bounded guidance-only changes
default to Quick, and material simplification triggers a depth/staffing reassessment.
Plan remains a job and can be brief notes. All behavioral Spec changes preserve
explicit Given/When/Then scenarios.

One accountable Engineer integrates software work. A separate clean-context
Reviewer in the same host owns the read-only verdict over the integrated
candidate. QA owns distinct Acceptance evidence; instruction-only work with no
separate outcome to exercise reuses Review and deterministic checks by reference.
Use professional instructions and the concrete workflow assignments; do not
require provider/model diversity, unused specialists, or one gate per Worker.

## Boundaries

- Current human instruction and recorded decisions own intent.
- Standing Spec plus approved delta owns durable behavior.
- Repository harness and accepted design/technical decisions govern execution.
- Findings, code, tests, logs, prototypes, and knowledge evidence cannot create
  requirements.
- Direct Explore, Spec, Plan, Review, Acceptance, Finish, Spec apply/merge, and KB work stop at
  the requested boundary.
- Importing a ticket or producing a mock does not authorize Build.
- Review PASS and local checks do not establish Acceptance.
- UI acceptance uses actual browser interaction and accepted visual comparison.
- Synthetic users stay within accepted scope and do not simulate research authority.
- Spec records approved meaning without routine canonical writes. Explicit Spec
  apply owns one proportional adversarial read plus existing guarded memory
  mechanics; routine Finish invokes that operation once at actual work completion.
- Ship executes only authorized publication and concise complete-candidate closure.

Fix aligned P0 and pragmatic P1 findings through coherent whole-outcome repair.
P2 does not extend the loop. Apply the shared three-cycle/three-substantive-repair
rethink trigger across Review and Acceptance, then stop automatic repair if the same
failure recurs after independent challenge.

## Ownership

- `skills/forge/SKILL.md`: public routing and lifecycle composition.
- `skills/forge-code-review/SKILL.md`: standalone Code Review leaf and portable
  direct-review boundary.
- `skills/forge/references/judges.md`: applicable judge dimensions, selection,
  replacement, disable, waiver, and report integration guidance.
- `skills/forge/references/`: phase, authority, runtime, host, and memory guidance.
- `skills/forge/assets/`: concise human-facing managed-document templates.
- `skills/forge/references/design-studies.md` and `assets/design-study.html`:
  codebase-grounded single-page studies; HTML is an ordinary linked visual asset,
  with revision and authority in the existing visual record.
- `agents/`: professional perspectives and temporary helper instructions.
- `src/` and compiled `dist/forge`: strict TypeScript/Bun mechanics with embedded templates.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brightstack/forge](https://github.com/brightstack/forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
