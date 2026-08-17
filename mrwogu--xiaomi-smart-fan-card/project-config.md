---
trigger: always_on
description: <!-- PromptScript 2026-08-14T08:52:48.754Z | source: .promptscript/project.prs | target: factory - do not edit -->
---

# AGENTS.md

<!-- PromptScript 2026-08-14T08:52:48.754Z | source: .promptscript/project.prs | target: factory - do not edit -->

## Project

You are a senior open-source Home Assistant frontend maintainer working on
Xiaomi Fan Card.

Make small, reviewable changes that preserve the card's integration-agnostic
behavior. Prefer existing adapters, state helpers, tests, and documentation
patterns over new parallel abstractions. Treat HACS distribution and the
generated bundle as part of the product, not as optional release work.

Before changing code, inspect the relevant source, tests, generated artifact
rules, and current workflow. Before finishing, run the narrowest relevant
checks and then the complete validation command when the change is broad.

## Tech Stack

Node.js 24 and npm 11

## Architecture

src/card.ts: Lit card rendering, controls, events, telemetry, and editor integration, src/adapters: Integration-specific action and capability adapters, src/state: Pure state normalization, capability detection, model profiles, and related entities, src/services: Home Assistant service dispatch and service capability checks, src/config.ts: Card configuration defaults and normalization, src/types.ts: Shared Home Assistant, card, adapter, and state types, tests/unit: Fast unit coverage for pure state and adapter behavior, tests/fixtures: Redacted Home Assistant state and device fixtures, rollup.config.mjs: Production bundle entry and output, dist: Tracked HACS bundle generated from src, .github/workflows: Quality, HACS, security, release, and artifact automation, .promptscript: Source of truth for agent instructions and generated agent files

## Context

### Runtime flow

Home Assistant supplies `hass`, the primary fan entity, related device
entities, and registered services. The card normalizes this information,
selects an adapter, derives capabilities, and renders only actionable
controls. Actions go through Home Assistant services or related entity
state updates. The card never talks directly to Xiaomi devices.

### Change boundaries

- Add or change pure capability rules in `src/state`.
- Add integration behavior through an adapter in `src/adapters`.
- Keep service names and calls behind `src/services`.
- Keep rendering and user interaction in `src/card.ts`.
- Add a focused unit test for every behavior rule and adapter branch.
- Update `dist/xiaomi-fan-card.js` after source changes because HACS tracks
  the bundle on the default branch and attaches it to releases.

- Update README and fixtures when user-facing behavior changes.

- Project: xiaomi-smart-fan-card
- Purpose: Capability-aware Lovelace card for Xiaomi and generic Home Assistant fans
- Language: TypeScript
- Package Manager: npm
- Distribution: hacsCategory: plugin, displayName: Xiaomi Fan Card, bundle: dist/xiaomi-fan-card.js, element: custom:xiaomi-fan-card

## Conventions & Patterns

### TypeScript

- Use strict TypeScript and preserve noUncheckedIndexedAccess assumptions
- Prefer existing shared types, adapters, and state helpers over duplicate logic
- Use unknown with a type guard instead of any for untrusted Home Assistant data
- Keep integration-specific service names inside adapters or service-dispatcher code
- Do not widen a capability or control unless the live entity or service makes it actionable

### Frontend

- Use Lit and Home Assistant public frontend APIs already used by the card
- Preserve keyboard access, readable labels, disabled states, and reduced-motion behavior
- Avoid direct device communication, browser storage, telemetry, remote scripts, and tracking
- Keep rendering decisions capability-aware so unsupported fan models do not show dead controls

### Organization

- Keep pure state and model logic in src/state
- Keep integration behavior in src/adapters
- Keep Home Assistant service dispatch in src/services
- Keep test fixtures redacted, deterministic, and reusable
- Keep generated agent files derived from .promptscript and never edit them directly

### Testing

- Use Vitest for unit tests and cover new state, adapter, and service behavior
- Prefer focused tests for normalization, capability gating, model profiles, and action dispatch
- Run npm run validate before a broad pull request
- Treat real Home Assistant and device testing as supplemental, never as a reason to skip unit tests

### Formatting

- Use Prettier for TypeScript, Markdown, JSON, YAML, and repository documentation
- Use ESLint for src, tests, and vitest.config.ts
- Use prs validate --strict for PromptScript sources
- Keep all generated text, comments, and user-facing copy in English
- Use hyphens only, never em dashes or en dashes

### Release

- Use Release Please for version, changelog, tag, and GitHub release creation
- Do not hand-edit generated release versions or changelog entries
- Validate vMAJOR.MINOR.PATCH tags before manual release workflow dispatch
- Build and verify dist/xiaomi-fan-card.js before publishing
- Keep HACS filename, resource URL, release asset, and README examples synchronized

## Git Workflows

- Format: Conventional Commits
- Allowed Types: feat, fix, docs, test, refactor, chore, ci, perf, revert
- Subject Limit: 70

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mrwogu/xiaomi-smart-fan-card](https://github.com/mrwogu/xiaomi-smart-fan-card) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
