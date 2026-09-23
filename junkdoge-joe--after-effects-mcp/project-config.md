---
trigger: always_on
description: These rules apply to human developers and coding agents working in this repository. They exist to keep engineering effort aligned with observable After Effects functionality. When an issue plan, reviewer suggestion, or local preference conflicts with these rules, stop and resolve the conflict in favor of the user-visible acceptance outcome unless the user explicitly changes the priority.
---

# Repository Development and Delivery Rules

These rules apply to human developers and coding agents working in this repository. They exist to keep engineering effort aligned with observable After Effects functionality. When an issue plan, reviewer suggestion, or local preference conflicts with these rules, stop and resolve the conflict in favor of the user-visible acceptance outcome unless the user explicitly changes the priority.

## 0.0 Read the current architecture direction first

**Read [`docs/ARCHITECTURE_DIRECTION.md`](docs/ARCHITECTURE_DIRECTION.md) before planning any work.** It was approved 2026-08-15 and it changes what work is worth doing. The delivery discipline in the rest of this file — evidence, acceptance tiers, stop conditions, `.aep` lifecycle — remains fully in force. Only the *direction* moved.

Three standing decisions constrain new work until the direction document says otherwise:

- **The native AEGP plane is frozen.** Keep the `.aex` and its 23 primitives; add none. Do not open a new native capability package, do not extend `native-primitives.json`, and do not run the capability-package codegen pipeline. The two properties that justify the plane — exact rational time and generation-bound locators — are already built.
- **The former package server plane has been removed.** Do not add handlers,
  backends, schemas, or entry points under `packages/`; the MCP server lives in
  the CEP Node context (`plugin/host/`).
- **The provider layer is three channels** — claude CLI, codex CLI, opencode. Do not add a fourth backend adapter; the retired `universalProviderRoute` / `providerCapabilityProbe` / `codexResponsesRoute` facades are gone and must not return.

Two things are settled; do not re-diagnose them from scratch:

- `plugin/host/jsx-bridge.js` keeps the serialization lock across a timeout and drains the persistent engine with a sentinel before releasing it (#260); `plugin/host/jsx-bridge.test.js` asserts that a timeout keeps the lock. Do not reintroduce an early release.
- Claude Desktop uses the installed extension's dependency-free
  `host/stdio-shim.js` with the user's system Node. Claude Code uses the panel
  URL directly; no package-server installation flow is supported.

## 0. User authorization is the scope boundary

- The current user's explicit requested outcome and named deliverables are the authorization boundary. An Issue, Epic, milestone, checklist, review comment, repository rule, prior plan, branch, worktree, or sunk implementation is context or evidence only; none authorizes additional product work or delivery mechanisms.
- A release request authorizes only the named release assets and the smallest indispensable build, signing, verification, documentation, and publication steps. It does not implicitly authorize an installer, RuntimeManager, zero-environment onboarding, new CI or runner topology, private artifact service, distribution or update channel, repair/rollback/uninstall system, new trust or secret infrastructure, an additional OS, architecture, or After Effects version, or another user workflow.
- Before introducing any such item—or any other deliverable or workflow not named by the user—stop, describe its concrete user benefit, expected footprint and cost, and explicit non-goals, and obtain new explicit approval. Do not create an Issue, branch, worktree, scaffold, secret, workflow, or implementation for it while awaiting approval.
- Treat a prerequisite as indispensable only when the named outcome cannot be produced or truthfully verified without it. If an existing or bounded manual path can produce the approved assets, replacement infrastructure is follow-up work, not a prerequisite.
- Unless the user explicitly approved a package brief that anticipates the footprint, stop and reconfirm scope before touching more than 15 files or adding more than 500 non-generated, hand-written lines; stop sooner whenever the delivery model or user workflow changes. Report implementation, tests, workflow/infrastructure, configuration/schema/fixtures, documentation, generated files, and mechanical version changes separately, together with their committed, staged, tracked, or untracked state. Crossing a threshold is a scope alarm, not permission to continue.
- If the user narrows or changes direction, first bring any in-flight or possibly side-effecting operation to a safe, reconciled checkpoint, then immediately stop the superseded path, cancel further implementation, review, and CI for it, preserve unfinished work only as clearly labeled recoverable state, and provide a keep/drop inventory. Continue only the newly approved scope; sunk cost never justifies finishing the old path.
- Quality, security, testing, and acceptance rules constrain how approved work is delivered; they do not expand what is approved. Resolve any conflict by satisfying the approved outcome with the smallest compliant change or by asking the user.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JUNKDOGE-JOE/after-effects-mcp](https://github.com/JUNKDOGE-JOE/after-effects-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
