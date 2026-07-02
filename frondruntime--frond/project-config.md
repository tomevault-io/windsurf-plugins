---
trigger: always_on
description: <purpose>Public publishing surface for the Effect-powered Frond runtime packages.</purpose>
---

# AGENTS

<repo>
  <name>frond</name>
  <purpose>Public publishing surface for the Effect-powered Frond runtime packages.</purpose>
  <stack>Bun, TypeScript, Effect v4 beta, MobX, React, Biome.</stack>
  <package-manager>bun</package-manager>
</repo>

<operating-principles>

## Agent Stance

- High signal, low noise. Every sentence must carry information.
- Verify before asserting. Search the workspace before claiming an API shape, file path, package export, or command exists.
- Prefer existing structure. Follow local package boundaries, naming, error style, and docs layout before introducing new patterns.
- Keep scope tight. Do the requested work. Report unrelated issues; do not fix them without explicit approval.
- Use deterministic tools. Formatting, import order, type checks, tests, and line counts belong to tools.
- Separate fact from inference. Mark assumptions when a conclusion depends on incomplete evidence.

</operating-principles>

<workflow>

## Default Workflow

1. Read relevant files before editing.
2. Check root and package-local package.json scripts before running commands.
3. Prefer rg / rg --files for search.
4. Make the smallest coherent change that satisfies the request.
5. Re-read modified files.
6. Run the narrowest useful verification command.
7. Report what changed, what was verified, and any residual risk.

</workflow>

<release-flow>

## Release Flow

This repository uses release-please-style release automation. Release notes and version bumps are inferred from Conventional Commit messages in merged history, usually the squash merge title.

- Use `fix:` for patch releases.
- Use `feat:` for minor releases.
- Use `type!:` or a `BREAKING CHANGE:` footer for major releases.
- Use `docs:`, `test:`, `chore:`, `refactor:`, `build:`, or `ci:` only when no package release should be produced.
- Prefer scopes when useful: `core`, `react`, `build`, `ci`, `docs`, `release`.
- Make PR titles merge-ready Conventional Commit titles.
- Put release-note context in the PR body when the change should appear in a GitHub release.
- Release-please creates version/changelog/tag/GitHub-release artifacts; npm publication remains a separate local manual step.
- Never wire npm publish into CI without an explicit release-policy change. Do not add npm tokens or trusted publishing by default.
- Before publishing, run `bun run publish:npm:dry-run` locally. It builds, packs, installs the `.tgz` files in a clean Bun consumer, typechecks with NodeNext, and runs an ESM import smoke.
- Publish with `bun run publish:npm` locally from an interactive terminal so npm can prompt for 2FA.

</release-flow>

<commands>

## Commands

- Use bun.
- Do not use another package manager unless explicitly asked.
- Do not call raw binaries when an existing script covers the job.

</commands>

<architecture>

## Frond Direction

Frond keeps the MobX-facing public model and runs runtime execution through Effect.

- Runtime must enforce node identity, readiness, operation admission, liveness, cancellation, scope, and stale-commit rules.
- React consumers use node classes, useNode, useNodes, Suspense, ErrorBoundary, computed fields, and domain methods.
- Driver authors use Frond Driver.Async or Driver.Effect hooks through node specs.
- Runtime owns graph identity, dependency readiness, per-node serialization, cancellation, scope, release, telemetry, and command execution.
- Graph/runtime owns node construction, lifecycle state, attempts, eviction, and liveness demand records.
- Node owns MobX domain state, computed getters, domain methods, and observation-derived liveness signals.
- MobX and React are adapters. They may mirror, subscribe, schedule through handles, and translate state for consumers. They must not own readiness or driver liveness truth.
- Node-to-node dependencies remain Frond graph edges. Effect Layer is for runtime services and driver dependencies, not a replacement for keyed node identity.

## Package Boundaries

- packages/core owns @frondruntime/core.
- packages/react owns @frondruntime/react.
- Keep this repository focused on public runtime packages and their supporting release/test tooling.
- No package may import from another package private source path.
- Runtime, graph, driver, node, keys, and MobX core must not import React or React adapter modules.
- Keep package barrels thin. Package index.ts files expose public surface; they must not hide runtime behavior.

## Hard No-list

- Do not use React presence as driver liveness.
- Do not mutate GraphNodeCell state from callback bridges unless the mutation is serialized through the node cell actor.
- Do not merge Effect service DI with Frond node spec overrides. They solve different dependency problems.
- Do not preserve obsolete APIs by default. Back compatibility is not required unless explicitly requested.

</architecture>

<code-policy>

## TypeScript And Effect

- Keep public fallible runtime boundaries as Effect where composition, typed failure, scope, or concurrency matters.
- Keep React/UI consumption Promise/MobX-native unless explicitly designing an Effect app surface.
- Prefer strict discriminated unions for domain states, protocol packets, outcomes, and lifecycle phases.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [frondruntime/frond](https://github.com/frondruntime/frond) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
