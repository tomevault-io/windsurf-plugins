---
trigger: always_on
description: transforms over plain data. _Why:_ cacheable, portable, testable, reasonable, parallel — purity
---

# AresRPG repository working agreement

AresRPG is a fully on-chain voxel MMORPG. This repository contains the game contracts, pure math,
SDK, indexer, realtime server, protocol, deterministic fight twin, frontend, engine, and authored
content.

## Read order

1. Read `ARCHITECTURE.md` completely. It is the current system law.
2. Search `DECISIONS.md` for the domain being changed. It contains active rulings and motives.
3. For TypeScript work, obey the [Code law](#code-law) below; ESLint is its executable floor.
4. For content or deployment operations, read `CONTENT_UPGRADES.md`.

Do not restate architecture in another document. Amend `ARCHITECTURE.md` when the current system
changes. Amend or delete an existing `DECISIONS.md` row when a ruling changes; never retain an
overruled row.

## Working method

- Keep one active objective. Do not let adjacent discoveries expand it.
- Work inline by default. Delegation is for explicitly requested, independent inspection or
  bounded execution; one agent retains judgment and verifies the result.
- Start from the owner of the fact, not the nearest call site. Search before creating a function,
  constant, table, state field, packet, store, cache, or dependency.
- Consider two seams and choose the one with fewer concepts and the smaller blast radius.
- Model data before logic. Prefer a rule that removes a special case over another branch.
- Write one premortem: `this fails if ...`; mitigate the likely answer first.
- New surfaces need an explicit design pass. A fix inside an established owner does not.
- Dependencies are permanent commitments. Prefer a small local implementation for a small need.

## Implementation law

- Pure transforms over plain immutable data are the default. Effects live at lifecycle edges.
- One reducer owns each stateful domain. Async results re-enter as inputs; callbacks never write
  stores.
- One fact has one owner. Import or derive it everywhere else; synchronized copies are forbidden.
- Do not abstract before a second concrete use. Straightforward code beats compact cleverness.
- Developer-chosen bindings use `snake_case`; PascalCase is for components and types;
  SCREAMING_SNAKE_CASE is for constants. React hooks remain `useX`.
- No classes or `this`, except React error boundaries, `extends Error`, and Three.js
  `extends PhysicalLightingModel`.
- Prefer cognitive complexity at most 10 and cyclomatic complexity at most 8. New code never
  exceeds 15/12, inherited hotspot scores never rise, nesting never exceeds 4, and files stay at
  most 600 lines. Extraction without removing decisions is not a complexity reduction.
- Await promises or explicitly discard them with `void`. Never swallow a failure.
- Handle discriminated unions exhaustively.
- Every ESLint disable names the rule and gives the local reason.
- Tests live under the package's sibling `test/` tree and mirror the source path.
- Wire/BCS decode tests pin at least one real captured payload with object/version/date provenance.

Preserve unrelated work in a dirty tree. Never reset, overwrite, or broadly reformat user changes.

## Debugging

1. Reproduce the reported symptom.
2. Read the complete error twice.
3. Inspect real inputs, process state, logs, chain state, and persisted projections.
4. Change one variable per probe and binary-search the state space.
5. After two failed fixes, stop patching and rebuild the model from evidence.
6. Do not ship a fix that cannot be explained mechanically.

When stuck, compress the problem into `SYMPTOM`, `REPRO`, `TRIED`, `HYPOTHESIS`, and the smallest
question that would close the evidence gap.

## Definition of done

- A bug is red-first: retain a deterministic regression reproducing the reported reason.
- Correctness intent becomes a fast machine-runnable check. Taste intent remains a human review
  checkpoint.
- Drive real behavior when practical. Exercise sad paths before happy paths.
- Review the final diff for stale consumers, dead exports, duplicate facts, security problems, and
  unexplained concept growth.
- Completion claims cite direct evidence: loaded file, fired request, transaction digest, log, or
  gate output.
- Treat net production lines as a review input. Refactors should normally be non-positive.
- Never automatically retry a transaction that executed and returned a digest.

## Project conventions

- Frontend state: Zustand modules, no prop drilling.
- Frontend UI: functional React and Tailwind utilities; no component library.
- Visual language: shared dark-purple surface tokens, gold/cyan semantic accents, JetBrains Mono,
  uppercase micro-labels, sharp terminal structure, and restrained atmospheric motion.
- Every player-facing string ships in all supported YAML locales under
  `packages/frontend/src/i18n/locales/` in the same change.
- `seed/` is the only authored content home. Do not add frontend content copies.
- The frontend never imports `@mysten/*`; every client-side chain write goes through the SDK.
- Commits use conventional subjects, bodies no longer than five lines, and one atomic concern.
- Changelog entries are player-facing release copy. Follow `CONTRIBUTING.md`'s audience law.

## Commands and gates

```bash
bun install
bun run dev
bun run lint
bun run typecheck

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aresrpg/aresrpg](https://github.com/aresrpg/aresrpg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
