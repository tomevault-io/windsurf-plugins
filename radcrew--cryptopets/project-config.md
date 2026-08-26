---
trigger: always_on
description: Root coordination contract for AI and human contributors in this repo. Detailed architecture and working guidelines live in [CLAUDE.md](./CLAUDE.md); this file states the non-negotiables and where to look.
---

# AGENTS.md

Root coordination contract for AI and human contributors in this repo. Detailed architecture and working guidelines live in [CLAUDE.md](./CLAUDE.md); this file states the non-negotiables and where to look.

## Scope

- Applies to the whole monorepo: `frontend/`, `backend/`, `mobile/`, `website/`, `shared/`, `contracts/ethereum/`, `contracts/solana/`, `services/indexer-go/`, `proto/`, `services/image-generator/`.
- No nested `AGENTS.md` files exist yet. If one is added under a package, it may tighten rules for that subtree but must not relax the rules here.

Normative language: `MUST`/`MUST NOT` are mandatory. `SHOULD`/`SHOULD NOT` are expected by default; deviations should be explained in the PR. `MAY` is optional.

## Non-Negotiables

- `MUST NOT` change Solana's frozen combat port (`game/battle_sim.rs`, `game/xp.rs`). It has no caller left in the program, but its golden-vector tests are what still prove `contracts/test-vectors/{battle,xp}.json` describe what actually settled on that chain. A bug found there is fixed forward in the live ports below, under a new `rulesetVersion`, never by patching the frozen one. **The Solidity port is gone**: `CombatSim.sol` was deleted once it had no on-chain caller, which also removed `battle.json`'s Solidity generator and validator. `battle.json` itself is unchanged and still gates the live ports.
- `MUST` keep the two **live** combat ports in step with each other and with the golden vectors: `protocol/src/combat/` (the canonical engine, re-exported from `shared/src/utils/combat` for existing importers) and `services/indexer-go/internal/combat/` (the independent verifier). Changing one without the other re-breaks the circuit breaker in §F, whose whole value is that the two were written to disagree if either drifts. This covers XP and level progression too (`protocol/src/combat/xp.ts`, validated against `contracts/test-vectors/xp.json`), so an XP or decay change is a both-ports change. `services/indexer-go/internal/combat/xp.go` still covers the formula and the decay but not level-up. It also covers **equipment modifiers** (roadmap §4): `protocol/src/combat/equipment.ts` and `services/indexer-go/internal/combat/equipment.go`, both validated against `contracts/test-vectors/equipment.json`. The modifiers apply at one specific point — after `extract`, before the skill multipliers — and moving that point in one port without the other changes every geared fight, so the ordering is pinned by a vector case in both.
- `MUST NOT` edit `contracts/test-vectors/{battle,xp,equipment}.json` to make a failing test pass — this holds more strongly now, not less. `battle.json` and `xp.json` are the only mechanical link left between the frozen ports and the live ones, and a live port that fails them has drifted away from the rules real battles were settled under. `equipment.json` is newer and has no frozen witness, but the same rule applies for the same reason: it is what holds the two live ports to one another. Its first case deliberately reproduces a `battle.json` row, so an ungeared fight is proven unchanged rather than assumed.
- `MUST` treat a `snapshot` or `ruleset` schema-version bump as append-only. An absent version means **1**, never "whatever this build implements": every snapshot and published bundle written before those fields existed is version 1 and has receipts signed over it, so defaulting to the current version silently re-encodes them under a layout they were never hashed under and invalidates every signature. Old versions stay listed in `SUPPORTED_VERSIONS` permanently (`protocol/src/domain/schemaVersions.ts`).
- `MUST NOT` assume the `ChainAdapter` interface (`shared/src/hooks/adapters/`) covers more than pet-action mutations and reads. It is a real, shared interface (`useEvmAdapter`/`useSolanaAdapter` both implement it) and every public pet-action hook consumes it for the mutation, but the low-level chain wiring in `frontend/src/chains/{ethereum,solana}/`, the async breed/mint randomness flows, and the combat simulator remain intentionally separate per chain. `useCreatePet` and `useBreedPets` are only chain-blind on the action: both carry the EVM settle lifecycle inline behind `isEvm` guards. See CLAUDE.md's cross-chain interfaces section for the exact boundary.
- `MUST` match the license of the package being edited when adding new files: `contracts/ethereum`, `contracts/solana`, `services/indexer-go`, `proto`, `protocol`, and `verifier` are MIT; everything else, `services/image-generator` included, is PolyForm Noncommercial 1.0.0 (root `LICENSE`). See the table in `README.md`. `protocol` is MIT on purpose (third parties have to be able to replay signed battle receipts), so it `MUST NOT` import from a PolyForm package; a test in that package enforces it. `verifier` is MIT for the same reason and depends on nothing but `protocol`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [radcrew/cryptopets](https://github.com/radcrew/cryptopets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
