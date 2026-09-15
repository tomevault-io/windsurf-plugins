---
trigger: always_on
description: This tree is the GrimCult fork: a GPLv3 Grim 2.0-modern base hosting the ported
---

# Codex Instructions

This tree is the GrimCult fork: a GPLv3 Grim 2.0-modern base hosting the ported
non-PacketEvents networking stack, the Java 3.0 simulation engine, and the Bedrock
simulation engine. The engine and anticheat rules below are the copyright holder's
own rules, carried over verbatim from the source repository; only the repository
layout and the build/test commands are adapted to this tree.

## Repository layout and build/test commands

- `common/` — platform-agnostic code: the raw-NMS networking stack (`network/`),
  packet event listeners (`events/`), the Java 3.0 simulation engine
  (`checks/impl/prediction/`, engine `utils/`), the Bedrock simulation engine
  (`bedrock/`), checks, managers, and the platform SPI (`platform/api/`).
- `bukkit/` — the Bukkit/Paper platform module. Bukkit/Paper-only; the Fabric
  modules were removed in the port. The distributable jar is built here.
- `legacy-placement-adapter/` — the pre-1.13 placement adapter, merged into the
  bukkit shadow jar.
- Full build: `./gradlew build` (output at `bukkit/build/libs/`)
- Unit tests: `./gradlew :common:test`
- Offline Bedrock replay tests: `./gradlew :common:offlineBedrockReplayTest`
- Live smoketests (real Paper server + real client): `scripts/run-local-all-smoketests.sh`

## Rules applicable to both the Java and Bedrock simulation engine

- For broader Grim compensation, packet timing, transaction, bundle, threading, and check-writing guidance, consult `CULT_ANTICHEAT_DEVELOPMENT_GUIDE.md`. If it conflicts with the Bedrock-specific uncertainty limits in this file, `AGENTS.md` wins.
- Assume cheaters will read Grim source code and look for bypasses, so exemptions and adding new lenience are not acceptable solutions.
- Temporary diagnostics may be added while investigating, but they must be removed before final validation or handoff.
- Applying changes to the check level is almost certainly wrong. The post prediction checks should be stupidly simple. No giving uncertainties due to being near certain blocks or otherwise working around simulation engine bugs. Instead, almost all changes should be made to the simulation engine itself, not post prediction checks.
- Model client packet processing at tick boundaries: packets are not processed in the middle of a client tick, so do not add half-tick uncertainty for packet arrival. Use tick-end and transaction ordering to prove when the client has processed state. Both Java and Bedrock clients guarantee packets in order.

When validating anticheat fixes, continue running the relevant smoke tests after each source-proven fix until the requested test target passes without false flags.

## Bedrock simulation engine changes

The Bedrock engine is a machine-written port of the Java simulation engine for Bedrock clients. It is required to be architected at the same high level as the Java engine:
- All Bedrock small-scenario fixes must validate to <= 0.001 block offset unless a tighter source-proven bound is required.
- Java engine architecture must be followed. The Bedrock engine should follow the Java engine's high-level runner architecture. The shared runner uncertainty handlers `PistonShulkerPush`, `CollisionModifier`, `StepTransform`, `Fireworks`, and `InsideBlock` are explicitly allowed for Bedrock, provided they operate on compensated state and exact Bedrock simulation/collision candidates. Other Java uncertainty classes must not be applied to Bedrock unless explicitly requested. The Java simulation engine is your source of truth for how to structure a simulation engine because it is carefully architected, while the Bedrock engine was machine-written.
- Stepping must be architecturally integrated with the Java runner, but the Bedrock engine must supply its own semantics.
- Collisions must create the same CollideAxisData and run NO FURTHER COLLISIONS within the engine other than for stepping, or Java-equivalent collision checking such as with sneaking. For example, running a collision downwards as the post position the player is at is not allowed.
- Do not use authored/provided client inputs as truth. The engine should derive possible movement first, then evaluate whether required input is legal at the post-prediction stage, similar to Java Grim.
- Sprinting legality belongs at the same level as input legality. The engine may model server-observable sprint state, but must not add sprint lenience to compensate for uncertain input.
- Packet delta represents future velocity. Do not move sprinting or movement multipliers into simulation if they belong to final check-side input allowance.
- Do not touch the evaluator unless the bug is proven to be there.
- Keep one system for each mechanic. Do not add duplicate sprint multipliers, duplicate evaluator lenience, or parallel ad hoc movement paths.
- When implementing movement logic within the Bedrock simulation engine, you MUST match the vanilla Bedrock client/server movement behavior as verified by official protocol documentation, packet/tick ordering, and replay validation. Do not run systems out of order unless it would violate other rules written in this document.
  * For Bedrock anticheat logic changes, cite the vanilla behavior being matched and the replay evidence that proves it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CultAC/CultAC](https://github.com/CultAC/CultAC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
