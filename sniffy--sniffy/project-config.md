---
trigger: always_on
description: These rules apply to every change in this repository. More specific `AGENTS.md` or `AGENTS.override.md` files may add
---

# Repository agent instructions

These rules apply to every change in this repository. More specific `AGENTS.md` or `AGENTS.override.md` files may add
constraints for their subtrees.

## Task ownership and autonomy

- Treat the linked GitHub issue and its acceptance criteria as the source of truth. Read the complete issue, relevant
  comments, existing pull requests, and current code before editing.
- For tasks explicitly marked ready for autonomous execution, do not stop for routine design choices or ask for
  confirmation. Inspect the repository, choose the smallest maintainable solution consistent with the issue, and record
  material decisions in the pull request. Stop only for a real blocker such as missing credentials, unavailable external
  infrastructure, destructive ambiguity, or mutually incompatible acceptance criteria.
- Complete the task end to end when permissions allow: implement, add or update tests, run applicable checks, review the
  diff, commit, push, and update or open a pull request. Use a draft while implementation or locally applicable
  verification is still in progress; unless the task explicitly requires a draft handoff, mark it ready for review after
  the implementation and those checks are complete. Never merge a pull request unless the task explicitly says to do so.
- Treat GitHub remote state as the publication source of truth. A local branch, local commit, `make_pr` metadata, or a
  final summary does not prove delivery. Before reporting publication, verify a resolvable remote branch, full commit SHA,
  pull-request URL, base/head branches, draft state, and matching head SHA through GitHub.
- Preserve unrelated user changes. Do not reset, clean, force-push, rebase shared branches, or rewrite history unless the
  issue explicitly requires it. Prefer a new `agent/<short-description>` branch from `develop` for new work.
- Before editing a task that combines two or more high-risk axes—new public APIs or artifacts, global mutable state,
  resource lifecycle/failure composition, multiple JDK/framework versions, or cross-reactor module placement—confirm
  that the issue contains a short design preflight: scope boundaries, module ownership, a failure/lifecycle matrix, and
  an acceptance-criterion-to-proof matrix. If a material product or public-API decision is unresolved or contradictory,
  stop before implementation and report that decision instead of inventing a broad compatibility layer.
- Before routing substantial work to Codex Cloud or unattended local Codex, apply the risk-axis and proof-matrix rules in
  `docs/retrospectives/2026-07-18-docusaurus-cloud-vs-local.md`. Prefer local execution for first-of-kind architectural or
  visual patterns and Cloud for isolated, fully specified tasks with deterministic Cloud-available proof. Do not estimate
  task suitability from changed-line or lockfile size alone.

## Standard tooling and infrastructure approval

- For established engineering domains such as dependency scanning, vulnerability management, formatting, linting,
  build orchestration, release automation, test reporting, coverage, packaging, and scaffolding, prefer maintained,
  widely adopted tools, official actions, platform features, and ecosystem-standard declarative configuration. Compose
  and configure those capabilities before writing custom code. Do not recreate capabilities already provided by npm,
  Maven, GitHub, Dependabot, GitHub Dependency Review, OSV-Scanner, CodeQL, Renovate, established linters/build systems,
  or equivalent maintained tooling.
- Evaluate maintenance status, ecosystem adoption, update path, security and permissions model, portability, expected
  failure/noise semantics, and operational burden—not only whether custom code can be made to pass tests.
- Bespoke build, CI, release, dependency, or security infrastructure requires a maintainer-approved rationale in the
  authoritative issue before implementation. It must state: the exact requirement and observable gap; maintained
  alternatives evaluated; why configuration or composition is insufficient; the smallest custom surface; owner and
  maintenance burden; security and permissions model; deterministic test strategy; upgrade and compatibility strategy;
  operational failure/noise semantics; and the removal condition and exit or migration plan. Missing approval is a
  blocker, not permission to invent a framework.
- Every new CI workflow or materially new job additionally requires the issue to document why an existing workflow or
  job cannot host the check; its trigger model; least-privilege permissions; required-check or blocking semantics;
  expected signal and acceptable noise; the maintainer response to failure; owner and lifecycle/removal condition; and
  overlap with existing GitHub or platform signals.
- Review architecture and operational simplicity before accepting green CI. Ask whether common-problem custom code is
  being introduced, which standard tools were considered, whether declarative composition can achieve the outcome,
  whether the custom surface is proportionate, and who maintains it as upstream formats, APIs, or advisories change.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sniffy/sniffy](https://github.com/sniffy/sniffy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
