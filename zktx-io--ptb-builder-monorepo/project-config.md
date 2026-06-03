---
trigger: always_on
description: PTB Builder is a React/TypeScript toolkit for building, editing, inspecting, and rendering Sui Programmable Transaction Block data.
---

# AGENTS.md

## Product Purpose

PTB Builder is a React/TypeScript toolkit for building, editing, inspecting, and rendering Sui Programmable Transaction Block data.

The package exists to make PTB structure easier to author and review:

- Edit PTBs through a graph UI.
- Convert between graph data and transaction-oriented data.
- Generate TypeScript SDK transaction-building code.
- Decode or load supported Sui PTB inputs into local editable structures.
- Let the host application handle wallet connection, signing, simulation, and execution.

PTB Builder is not a wallet, custody layer, transaction safety guarantee, or autonomous executor. Do not describe it as one.

## Repository Map

Inspect the actual repository before assuming any path, script, package, or API exists. If the repository differs from this map, the repository is the current fact and this section should be updated when the difference is intentional.

Current structure:

- `packages/ptb-model/`: package `@zktx.io/ptb-model`; UI-independent PTB model, TransactionIR, raw PTB conversion, Mermaid renderer, and TS SDK code string renderer.
- `packages/ptb-cli/`: package `@zktx.io/ptb-cli`; CLI for converting local or read-only fetched PTB transaction data to Mermaid through `@zktx.io/ptb-model`.
- `packages/ptb-builder/`: published package `@zktx.io/ptb-builder`.
- `packages/example/`: local Vite example app.
- `.WORK/`: ignored local investigation and planning notes.

Current source areas inside `packages/ptb-builder/src/`:

- `ptb/`: PTB document, graph/model adapters, registry, Core client bridge, and runtime `Transaction` adapter logic for host-owned simulation/execution paths. It must not sign, execute, or take custody.
- `ui/`: React builder UI and React Flow integration.
- `styles/`: package styles and theme exports.
- `types/`: local ambient type declarations for builder UI dependencies.

## Commands

Inspect `package.json` before running project commands. Do not invent scripts.

Current root commands, as of the current `package.json`:

- Install: `npm install`
- Build model package, CLI package, then builder package: `npm run build`
- Run example after build: `npm run dev`
- Run builder-flow tests sequentially: `npm run test:builder-flow`
- Test CLI package with test-source type checking: `npm run test:cli`
- Test model package: `npm run test:model`
- Lint: `npm run lint`
- Format: `npm run format`

Current package commands:

- Model build: `npm run build --workspace @zktx.io/ptb-model`
- Model type check: `npm run typecheck --workspace @zktx.io/ptb-model`
- Model test: `npm run test --workspace @zktx.io/ptb-model`
- CLI build: `npm run build --workspace @zktx.io/ptb-cli`
- CLI type check: `npm run typecheck --workspace @zktx.io/ptb-cli`
- CLI test source type check: `npm run typecheck:test --workspace @zktx.io/ptb-cli`
- CLI test: `npm run test --workspace @zktx.io/ptb-cli`
- Builder build: `cd packages/ptb-builder && npm run build`
- Builder type check: `cd packages/ptb-builder && npm run typecheck`
- Builder test: `cd packages/ptb-builder && npm run test`
- Builder lint: `cd packages/ptb-builder && npm run lint`
- Example dev: `cd packages/example && npm run dev`
- Example test: `cd packages/example && npm run test`
- Example build: `cd packages/example && npm run build`
- Example lint: `cd packages/example && npm run lint`

Never claim a test, build, lint, pack, or verification step passed unless it was actually run and observed successfully. If `package.json` differs from this section, `package.json` wins; say so and update this section when the difference is intentional. If a command does not exist, say so and use the closest available verification.

Do not run builder and example tests in parallel. Both flows consume package
build artifacts, and concurrent clean/build steps can remove `dist/` while the
other test runner is resolving workspace package exports. Use
`npm run test:builder-flow` for the sequential builder/example test gate.

## Communication Rules

- Answer with verified facts and concise conclusions.
- Do not waste tokens on excuses, filler, or unsupported speculation.
- State uncertainty plainly when evidence is incomplete.
- Separate facts, assumptions, and recommendations.
- When reporting mistakes, missed scope, incomplete work, regressions, false prior claims, or deleted behavior, state the concrete failure directly. Do not soften it with euphemisms such as "reduced," "partial," "scope adjustment," "rough edge," "tradeoff," or "follow-up" when the verified fact is that required work was not implemented, was removed, was narrowed, or was misrepresented as complete. These examples are non-exhaustive: any label used to soften, obscure, legitimize, or relabel a verified contraction is forbidden, including neutral-sounding labels such as "aligned," "normalized," "consolidated," "simplified," "tightened," "smaller," or "narrower" when they are used to avoid naming the contraction as a defect or deliberate removal.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zktx-io/ptb-builder-monorepo](https://github.com/zktx-io/ptb-builder-monorepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
