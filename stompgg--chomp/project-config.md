---
trigger: always_on
description: **Constraints first, then design.** Before proposing any solution, identify the hard constraints (language semantics, type system, inheritance, runtime behavior). If the approach conflicts with a constraint, don't propose it. Zero band-aid attempts — if it doesn't fit cleanly, the design is wrong. Redesign, don't force.
---

# CLAUDE.md

## How to Work

**Constraints first, then design.** Before proposing any solution, identify the hard constraints (language semantics, type system, inheritance, runtime behavior). If the approach conflicts with a constraint, don't propose it. Zero band-aid attempts — if it doesn't fit cleanly, the design is wrong. Redesign, don't force.

**Measure before deducing.** When debugging, add one targeted diagnostic and look at the data. Don't build chains of reasoning from assumptions about what the code "should" do. If the first theory doesn't match observations, measure — don't generate more theories from the same unverified premises.

**Fix at the right layer.** Don't patch symptoms. If a fix requires callers to know implementation details, it's at the wrong layer. If the same pattern needs 3+ special cases, the abstraction is wrong.

## Project Overview

**C.H.O.M.P.** (Credibly Hackable On-chain Monster PvP) is an on-chain turn-based PvP battling game inspired by Pokemon Showdown and M.U.G.E.N. Built on Solidity using the Foundry framework, it features an extensible battle engine where users can create custom moves, monsters ("mons"), effects, abilities, and hooks.

**License:** AGPL-3.0
**Solidity version:** 0.8.34

## Quick Start

```bash
forge install        # Install dependencies (forge-std)
forge build          # Compile contracts
forge test           # Run all tests
forge test -vvv      # Run tests with verbose output
```

## Repository Structure

```
chomp/
├── src/                    # Solidity source contracts
│   ├── Engine.sol          # Core battle engine (main entry point)
│   ├── IEngine.sol         # Engine interface
│   ├── Structs.sol         # All shared data structures
│   ├── Enums.sol           # All shared enums (Type, MoveClass, EffectStep, etc.)
│   ├── Constants.sol       # Global constants (move indices, defaults, sentinel values)
│   ├── DefaultValidator.sol # Validates game rules (team sizes, move legality, timeouts)
│   ├── DefaultRuleset.sol  # Configures initial global effects for battles
│   ├── IValidator.sol      # Validator interface
│   ├── IRuleset.sol        # Ruleset interface
│   ├── IEngineHook.sol     # Hook interface for battle lifecycle events
│   ├── abilities/          # Ability interface (IAbility.sol)
│   ├── commit-manager/     # Commit-reveal scheme for simultaneous moves
│   │   ├── DefaultCommitManager.sol
│   │   ├── SignedCommitManager.sol   # EIP-712 signed commits
│   │   ├── SignedCommitLib.sol       # Shared signed-commit helpers
│   │   └── ICommitManager.sol
│   ├── cpu/                # AI opponents (CPU players)
│   │   ├── CPU.sol                # Base CPU
│   │   ├── HeuristicCPUBase.sol   # Shared heuristic scaffolding for BetterCPU / FairCPU
│   │   ├── BetterCPU.sol          # Smarter AI
│   │   ├── FairCPU.sol            # Balanced opponent
│   │   ├── OkayCPU.sol            # Mid-tier opponent
│   │   ├── CPUMoveManager.sol     # Wraps Engine.execute for CPU-driven battles
│   │   └── ICPU.sol
│   ├── effects/            # Effect system (status effects, battlefield)
│   │   ├── IEffect.sol     # Effect interface with lifecycle hooks
│   │   ├── BasicEffect.sol
│   │   ├── StaminaRegen.sol
│   │   ├── status/         # Status effects (Burn, Frostbite, Panic, Sleep, Zap) + StatusEffectLib
│   │   ├── battlefield/    # Battlefield effects (Overclock)
│   │   # NOTE: stat boosts are inlined into the Engine (see "Stat Boosts" below); the math
│   │   #       helpers live in src/lib/StatBoostLib.sol, there is no StatBoosts effect contract.
│   ├── game-layer/         # Team / mon registry, gacha, exp, facets, quests, gifts
│   │   ├── GachaTeamRegistry.sol   # Concrete leaf: composes the abstracts below
│   │   ├── MonOwnership.sol        # monsOwned set + ownership view/check helpers
│   │   ├── MonRegistry.sol         # Owner-managed mon catalog (stats / moves / abilities / metadata)
│   │   ├── PlayerProfile.sol       # Packed playerData slot, CPU/whitelist flags, assigner allowlist
│   │   ├── PackedTeamStore.sol     # Bit-packed team CRUD (4 teams per slot, 16 slots per player)
│   │   ├── Facets.sol              # 12-facet ± stat tradeoff system (abstract)
│   │   ├── MonExp.sol              # Packed exp, level curve, level-up facet draws, assignExp
│   │   ├── Quests.sol              # Daily quest pool + packed predicate evaluator (abstract)
│   │   ├── ReturnerGift.sol        # Daily-return points + exp gift contract (assigner)
│   │   ├── ITeamRegistry.sol       # Public read/write surface that the leaf exposes
│   │   ├── IGachaPointsAssigner.sol / IExpAssigner.sol  # Owner-allowlisted off-band grants
│   │   └── IPhantomTeamRegistry.sol  # CPU-relayer entry for writing user phantom team configs
│   ├── hooks/              # Engine hooks (e.g. SimplePM)
│   ├── lib/                # Utility libraries: ECDSA, EIP712, Ownable, EnumerableSetLib,
│   │                       # MappingAllocator, MerkleProofLib, Multicall3, StaminaRegenLogic,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stompgg/chomp](https://github.com/stompgg/chomp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
