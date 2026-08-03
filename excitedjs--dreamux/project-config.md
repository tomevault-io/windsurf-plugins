---
trigger: always_on
description: Always loaded. Keep this file to guardrails that must be present in every
---

# dreamux repository operating rules

Always loaded. Keep this file to guardrails that must be present in every
agent context. On-demand architecture and reference material lives in
[`.agents/root.md`](.agents/root.md).

When this file and `.agents/` disagree, this file is authoritative for operating
rules. For architecture facts, read the current source first, then the linked
KB entry.

## Architecture Discipline

Refactoring is never "done" — it is always on the road. Treat the architecture
as a living thing every task must leave at least as clean as it found it.

- **Refactor-first, not glue-first.** Before implementing any non-trivial
  requirement, ask: *does this need an architecture change, or a new/extended
  capability?* If the clean home for the logic does not exist yet, create or
  reshape it. Do **not** stitch behavior together with ad-hoc glue (magic-string
  prefixes, target data smuggled through prompts/free-text fields, state
  re-derived in core that a lower layer already owns authoritatively,
  responsibilities bolted onto whatever class is nearest). Glue is how a
  codebase rots into a mess.
- **Self-check layering every time.** For each change, verify the module
  boundaries still hold: is this logic in the layer that owns it? Does it honor
  the neutral seams (core stays behind `AgentRuntimeProvider` / `ChannelProvider`;
  no runtime/channel specifics leak into shared/core layers)? Would a new
  capability/contract be the right home instead of a special case? If a change
  fights the layering, the layering — or the change — is wrong; stop and fix the
  boundary, don't force it through.
- **Prefer a capability over a special case.** When two features want the same
  underlying fact or action (e.g. "is the agent idle", "where does this egress
  go"), design one foundational capability the whole core depends on, rather
  than re-solving it ad hoc each time.
- **Leave the cleanup trail.** When a task reveals a boundary that should move,
  either fix it or record it (`.agents/`, an issue, or the knowledge-delta
  update) — never silently pile another layer of glue on top.
- **Do not weaken a load-bearing test to make a change pass.** Some tests encode
  a locked contract (e.g. the issue #63 non-blocking-inbound live gate). If a
  change makes such a test fail, the change is usually wrong — fix the change,
  not the assertion. When a diff edits a test's assertions, "the tests pass" is
  circular: review the test diff against the source contract, never trust a
  green run produced by a rewritten test. When reviewing a fix, review the whole
  current change holistically (a narrow "did it fix X" pass hides regressions
  the fix introduced).

The goal is explicit: this repo must not degrade into a spaghetti/"big ball of
mud" codebase. Cleanliness of module layering is a standing acceptance criterion,
not an optional nicety.

## Communication

- Reply to the user in **Chinese**.
- Write all repo docs, `.agents/` docs, code comments, commit messages, and PR
  descriptions in **English**.

## Current Source Of Truth

- Current architecture entry point: [`.agents/reference/current-architecture.md`](.agents/reference/current-architecture.md).
- Repository/package layout: [`.agents/reference/repo-structure.md`](.agents/reference/repo-structure.md).
- State/cache/run/log paths: [`.agents/reference/state-and-paths.md`](.agents/reference/state-and-paths.md).
- Channel/Feishu runtime: [`.agents/reference/channel-runtime.md`](.agents/reference/channel-runtime.md).
- Task routing and KB index: [`.agents/root.md`](.agents/root.md).
- KB writing rules: [`.agents/CONTRIBUTING.md`](.agents/CONTRIBUTING.md).

Before answering architecture/domain questions, verify against source code. The
KB explains intent and history; code is the current behavior.

## Build And Test

`excitedjs/dreamux` is a Rush + pnpm monorepo. Use the monorepo path only:

```bash
node common/scripts/install-run-rush.js update
node common/scripts/install-run-rush.js build
node common/scripts/install-run-rush.js test
```

Do not use per-package `npm install`; workspace dependencies use `workspace:*`.

## Always-Binding Rules

- **Public repo red line:** never commit internal identifiers, secrets, private
  registry URLs, internal hostnames, or real Feishu ids/tokens. `.gitleaks.toml`
  and `.npmrc` are shared canonical guardrails with the sibling repo; if a
  guardrail false-positives, stop and ask rather than editing a local allowlist.
- **No synchronous blocking IO in package source:** `/packages/*/src/**` must
  use async fs/process APIs. `rush lint` enforces the shared
  `@excitedjs/eslint-config` no-sync-IO gate.
- **No runtime dependencies on dev tools:** bin launchers execute compiled
  `dist/` with plain `node`; do not reintroduce `tsx`.
- **Launcher path handling:** new bin launchers must follow the
  `/packages/dreamux/bin/dreamux` symlink-walk shape.
- **Path contracts:** host-owned path builders belong in
  `/packages/dreamux/src/platform/paths.ts`; volatile runtime socket allocation
  belongs in `/packages/dreamux/src/platform/runtime-sockets.ts`; runtime-owned
  path derivation belongs in the runtime package that uses it (for example
  `/packages/agent-runtime/codex/src/paths.ts`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [excitedjs/dreamux](https://github.com/excitedjs/dreamux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
