---
trigger: always_on
description: Write code that will not need to be rewritten.
---

# Project Engineering Guide

Write code that will not need to be rewritten.

These rules apply to the project being worked on, regardless of its language, framework, or agent platform. Follow the current user's request and the project's established constraints; this guide does not grant additional authority.

## Read Before You Write

- Before changing a project, read its existing README and contribution guide (such as CONTRIBUTING.md), then any relevant scoped instructions. Follow the current project's commands and constraints.
- Read each file to be modified in full; an unchanged version already read in context satisfies this requirement. Inspect relevant callers, imports, configuration, dependencies, and tests before choosing a change boundary.
- Reuse confirmed project patterns. Explain the local reason and impact when changing an established interface, library, or structure.

## Think Before Coding

- State assumptions that affect behavior, data, scope, or risk. Resolve questions from project facts first.
- When unresolved interpretations materially change the result, explain the choices. Ask before high-risk or irreversible action; for low-risk ambiguity, state a reasonable assumption and how it will be checked.
- Analysis, review, diagnosis, and design requests do not authorize implementation writes. When implementation is requested, continue through relevant local verification and affected documentation without requiring repeated phase approvals.
- Existing authorization does not extend to unrelated changes, paid runs, external messages, deployments, or destructive actions.

## Simplicity and Scope

- Deliver the smallest end-to-end change that meets the current requirement. Do not add abstractions, configuration, fallback branches, or future extension points without a concrete need.
- Preserve unrelated user changes. Do not reset, overwrite, or reformat them.
- Keep each component's responsibility clear. Remove code, imports, tests, and documentation made stale by this change, but leave unrelated cleanup alone.
- When all callers are local and can be updated together, replace the old interface directly. Keep compatibility only for a confirmed external contract, such as a public API, persisted format, or third-party integration.
- When scope expands, split independent verifiable work or revisit the unresolved design instead of silently broadening the task.

## Avoid Overdefense

- Do not propose SHA, hash, content-fingerprint, or digest-binding schemes unless both conditions hold: they replace a materially more expensive operation, and their result changes what happens next.
- Do not re-read files already in context that are confirmed unchanged.
- Do not add defensive scaffolding: feature flags, migration frameworks, compatibility layers, or wrappers for cases that do not occur in the current project.
- Where judgment is needed, make the judgment. Do not replace it with a scoring table, checklist, or re-verification loop.
- Deliverable text is not a defense transcript. State plainly what holds; collect necessary caveats in one "Limitations" section, using the task's language.
- Do not write writing instructions into the deliverable. "Do not mention X" means X is absent, not that the deliverable says "we do not address X."

## Verification and Execution

- Define observable success criteria before implementation. For multi-step work, briefly pair each step with its verification; use native task tools only when useful.
- Test behavior, boundaries, and regression risks, not incidental implementation details. For a bug, construct a failing test or reproducible signal first when practical.
- Start with the smallest relevant check, then broaden according to impact. Discover actual project commands; do not assume a package manager or test runner.
- Use fresh evidence from the current worktree before claiming success. Report checks not run, unavailable reproduction, and remaining risks; old logs and another agent's summary are not proof.
- If tests are explicitly out of scope, use appropriate static checks, dry runs, read-back, or focused diff review and state what they cannot establish.
- Continue until the authorized goal is verified or a real blocker requires input. Do not rerun successful checks without new changes, failures, or unresolved doubts.

## Debugging and Dependencies

- Read the full error, relevant inputs, logs, and runtime conditions. Test one explicit hypothesis at a time when locating a failure.
- Repair the cause and preserve caller-visible failure semantics. Retries, swallowed exceptions, null checks, or default values are not substitutes for finding the cause.
- Before adding a dependency or reimplementing a common capability, inspect existing code, dependencies, standard-library support, and suitable maintained libraries.
- Verify capabilities against actual versions, callers, types, or official documentation. Explain any new dependency's concrete need and maintenance impact; update affected manifests, lockfiles, and deployment documentation together.

## Code Comments

- Explain non-obvious contracts, reasons, and consequences rather than narrating syntax.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tx1207/hello-scholar](https://github.com/Tx1207/hello-scholar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
