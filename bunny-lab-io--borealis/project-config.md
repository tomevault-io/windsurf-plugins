---
trigger: always_on
description: Respond like smart caveman. Cut all filler, keep technical substance.
---

## Rules of Engagement with Developer
Respond like smart caveman. Cut all filler, keep technical substance.

* Drop articles (`a`, `an`, `the`) and filler (`just`, `really`, `basically`, `actually`).
* Drop pleasantries (`sure`, `certainly`, `happy to`).
* No hedging. Fragments fine. Short synonyms.
* Technical terms stay exact.
* Code blocks unchanged.
* Pattern: `[thing] [action] [reason]. [next step].`

Use this file as entrypoint for Codex instructions. Full knowledgebase lives under `Docs/`, with navigation and documentation rules in `Docs/index.md`.

## Core Operating Model
Treat Borealis work as iterative repo loop, not single prompt.

Default loop:
1. Read `Docs/index.md`.
2. Read relevant domain docs.
3. Treat final `??? example "Detailed Codex Breakdown"` sections as authoritative Codex guidance for that domain.
4. Inspect issue, code, tests, runtime paths, and existing conventions.
5. Identify concrete goal and validation path.
6. Make smallest coherent change.
7. Run narrowest useful validation first.
8. Expand validation when change risk requires it.
9. Update docs, SBOM, or issues when change requires durable record.
10. Summarize changed files, verification, risks, and next step.

Do not call work complete because files changed. Work complete only when requested behavior is implemented, relevant validation has run or been explicitly deferred, and handoff includes remaining risk.

## Where to Read
* Start at `Docs/index.md`.
* Use index table of contents to find domain documentation, testing guidance, runtime docs, API docs, and operation runbooks.
* Follow domain docs found through index.
* Where docs overlap, domain page wins.
* `Detailed Codex Breakdown` admonitions inside each page are authoritative agent guidance.
* Do not duplicate domain guidance across multiple docs. Link to canonical page instead.

## Strong Goal Rule
Convert vague work into verifiable goal before implementation.

Weak goals:
* `Fix issue.`
* `Implement plan.`
* `Update docs.`

Strong goals include:
* Expected behavior.
* Affected subsystem.
* Relevant domain docs.
* Validation command or validation method.
* Docs, SBOM, migration, or technical-debt impact.
* Clear definition of done.

If ambiguity blocks safe implementation, ask operator. If ambiguity is minor, make reasonable assumption and state it in handoff.

## Definition of Done
For code changes, work is ready for review only when applicable items are true:
* Requested behavior implemented.
* Existing behavior preserved unless change explicitly requires breakage.
* Relevant unit tests, lint, build, runtime validation, or domain-specific checks performed.
* New or updated tests added when behavior changes and test path exists.
* Operator-facing docs updated when behavior, deployment, configuration, API, troubleshooting, or operational flow changes.
* `Docs/Reference/SBOM.md` updated when third-party software is added, removed, vendored, or downloaded.
* GitHub issue with `Technical Debt` label created or updated when patchy workaround, non-standard build step, or dev/prod divergence introduced.
* Handoff includes summary, changed files, verification, risks, and next step.

Never claim tests passed unless command actually ran and passed.

If validation cannot be run, state why and provide exact command or next steps operator should run.

## Durable Documentation Memory
`Docs/` is Borealis project memory.

Do not leave durable project knowledge only in chat. When Codex learns durable project knowledge, update correct repo surface.

Placement rule:
* Operator procedure -> visible operator-facing section in relevant `Docs/` page.
* API paths, source maps, runtime behavior, debug flow, implementation notes -> final `??? example "Detailed Codex Breakdown"` section.
* Workaround, non-standard build step, or dev/prod divergence -> GitHub issue labeled `Technical Debt`.
* Third-party software addition, removal, vendoring, or download -> `Docs/Reference/SBOM.md`.
* Repo-wide Codex behavior -> this `AGENTS.md`.
* Open implementation work -> issue or PR, not hidden chat context.
* Closed loop or resolved investigation -> relevant doc, issue comment, or PR summary.

Do not create separate scratch-memory docs when domain doc, issue, PR, or SBOM entry is correct durable surface.

## Documentation Authoring Style
* Write operator-facing docs like `Docs/Engine/deploying-the-engine.md`: short opening explanation, clear requirements, normal path first, then first-run checks or verification.
* Do not add visible `Purpose` section. Put plain-language purpose directly under page title so page starts quickly.
* Keep visible sections friendly and task-focused. Explain what operator should do, what operator should expect, and what can go wrong.
* Keep implementation detail out of operator path. API endpoints, related documentation, source paths, database tables, implementation notes, debug flow, and Codex-only reasoning belong inside final `??? example "Detailed Codex Breakdown"` section.
* Structure `Detailed Codex Breakdown` sections consistently:
  * `### API endpoints` when endpoint details matter.
  * `### Related documentation` for cross-links and reading order.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bunny-lab-io/Borealis](https://github.com/bunny-lab-io/Borealis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
