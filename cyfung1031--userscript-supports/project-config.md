---
trigger: always_on
description: This document is the canonical operating contract for coding agents working in this repository.
---

# Repository Agent Operating Contract

This document is the canonical operating contract for coding agents working in this repository.
It is written to be portable across GitHub repositories, programming languages, and toolchains.

In this contract, “PR” means the repository’s reviewable change proposal: pull request, merge
request, change request, patch review, or equivalent. “Final PR head” means the final reviewed
revision, commit, patch, changeset, or equivalent immutable review state.

This contract applies to every PR type, including feature requests, bug fixes, refactors,
performance work, tests, documentation, generated or mechanical changes, dependencies, build and
configuration changes, migrations, security and privacy changes, and release work. The common gates
apply to all of them; the change-type gates in §7 add relevant checks without narrowing coverage.

`CLAUDE.md` is a canonical alias of this file and must not contain a second policy. Its portability
claim is conditional on the consuming environment resolving the relative alias. If a loader cannot
resolve it, do not proceed on an assumed or duplicated contract: use a platform-supported include
that still renders this file as the sole policy source, or mark the work `NOT_READY` until the
canonical contract is loadable.

Before relying on an alias or platform include, record canonical-resolution evidence: the loader
path or mechanism, resolved `AGENTS.md` identity, and content hash or equivalent byte-identity check.
Also record the observed loader resolution graph, precedence, and final loaded bytes or hash. If the
actual consuming loader cannot be observed, mark policy loading `UNVERIFIED` and do not return
`READY`; a local filesystem check alone does not prove the runtime loader used the canonical policy.

## 1. Purpose and authority

The code and diff explain **what** changed. The pull request (PR) description must explain **why**
the change was necessary, what problem it addresses, and what evidence makes the decision sound.
Reviewers must not have to reconstruct the user's request or the agent's private reasoning from a
diff.

Apply this contract together with repository-specific instructions, contribution guidelines, and
maintainer decisions. A more specific repository rule may add requirements; it must not silently
remove this PR-rationale gate. Higher-priority system, platform, security, and explicit user
instructions remain authoritative.

This file records observable decision evidence. It must never be used to expose hidden chain of
thought, credentials, private user data, or other secrets. Document concise reasons, facts,
trade-offs, and verification—not private internal deliberation.

### Truthfulness and imperfection

No PR is perfect. A responsible PR makes remaining uncertainty visible instead of presenting the
selected solution as universally correct. Distinguish important claims as `PROVEN`, `INFERRED`,
`UNVERIFIED`, or `CONTRADICTED`, and state the consequence of each status. Report limitations,
regressions, residual risk, rejected evidence, and accepted trade-offs even when they make the PR
less persuasive.

Claim status is always bounded by the declared scope, revision, environment, oracle, and evidence:

- `PROVEN(scope)` means the evidence establishes the claim for that declared scope; it is not a
  universal guarantee.
- `INFERRED(scope)` means the claim is the best supported explanation but has not been directly
  established.
- `UNVERIFIED(scope)` means a decision-relevant check or oracle is missing; name the missing check
  and its consequence.
- `CONTRADICTED(scope)` means an authoritative or observed result conflicts with the claim; stop
  relying on the claim until the conflict is resolved or explicitly adopted as a changed premise.

An explicit user request can authorize an investigation or propose a scope, but it is not by itself
evidence that a problem exists or that implementation is necessary. If the request is weak,
ambiguous, invalid, or unsupported, say so. Ask for clarification, stop the PR, or reframe it as a
clearly labelled investigation or decision proposal. Never manufacture a stronger
problem statement from a weak request.

When a user request, maintainer decision, repository contract, test evidence, specification, or
runtime observation conflicts with another source, preserve the conflict explicitly. Identify each
source, its scope and authority, and the affected decision. Do not silently select the most
convenient source or convert an unresolved conflict into `READY`. A conflict is resolved only when
an identified, authority-verified owner or higher-authority maintainer explicitly supersedes one
source with another as the governing premise for a declared scope and decision, records the exact
source versions, superseded source, adoption rationale, acceptance evidence and trade-off, and
updates the affected acceptance and risk checks. Until then, mark it `CONTRADICTED` or `UNVERIFIED`
and keep the PR `NOT_READY`.

### Request appropriateness gate

Before coding, assess whether the requested work is appropriate to perform. If it is harmful,
unsafe, unauthorized, privacy-invasive, materially out of scope, incompatible with a governing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cyfung1031/userscript-supports](https://github.com/cyfung1031/userscript-supports) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
