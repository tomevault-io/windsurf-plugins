---
trigger: always_on
description: Shared guidance for AI coding agents and contributors. This is the canonical guide; `CLAUDE.md` imports it so Claude Code picks it up automatically, and other tools that read `AGENTS.md` get the same content. Navigation reference: where to find what, and the rules that must not break. For deeper context:
---

# AGENTS.md

Shared guidance for AI coding agents and contributors. This is the canonical guide; `CLAUDE.md` imports it so Claude Code picks it up automatically, and other tools that read `AGENTS.md` get the same content. Navigation reference: where to find what, and the rules that must not break. For deeper context:
- `CONTEXT.md`: domain glossary. Decodes the Minecraft parkour and movement vocabulary (facing, direction, velocity, tier, neo, momentum, byte-exact, etc.). Read it first; almost none of these terms are in an LLM's training data.
- `docs/reference/mcpk/`: in-repo mirror of the Minecraft Parkour Wiki physics (movement formulas, constants, the sine table, collision order, block friction, status effects, tiers). The byte-exact ground truth this tool replicates; if code disagrees with a number there, the code is the bug.
- `docs/VISION.md`: north-star goal (two blocks in, full TAS out), the capability arc, design principles
- `docs/CODING_GUIDE.md`: module rules, where new code goes, port pattern, per-module toolchains
- `docs/research/`: angle-solver design record, ILS global-solve notes
- `CONTRIBUTING.md`: feature definition checklist, Conventional Commits, release-please flow


## Modules

```
core/                  Java 8.  ImGui-only UI/data + the angle solver. No MC, Fabric, Forge, or LWJGL imports.
forge-core/            Java 8.  Shared for both Forge loaders: lwjgl2/ ImGui bootstrap, sim/ sprint machine. No MC imports.
loader-fabric-1.21.10/ Java 21. Fabric (Loom, LWJGL3). MC-touching sim/render/mixins/entry point. Source under src/client/java.
loader-forge-1.8.9/    Java 8.  Forge (Unimined FG2, LWJGL2). MC-touching code.
loader-forge-1.12.2/   Java 8.  Forge (Unimined FG3, LWJGL2). MC-touching code.
```

The two Forge loaders are intentional duplicates: 1.8.9 and 1.12.2 have incompatible MC APIs (`Vec3` vs `Vec3d`, `theWorld/thePlayer` vs `world/player`, `moveEntity` vs `move`, etc.). `forge-core` holds only the MC-free shared parts.


## Core flow

1. **Inputs** (core): `InputOverlay` edits `InputData` (list of `InputRow`, one per tick: W/A/S/D/Jump/Sneak/Sprint/L-click/R-click + yaw/pitch + amplifiers).
2. **Simulation** (loader): the loader's `Simulator` port drives a `SimulatorEntity` (a real MC player subclass) tick by tick, recording positions.
3. **Visualization** (core + loader): `BoxController` (core) stores positions and decides what to draw, then calls the loader's `BoxRenderer` port for the actual GL draw. Start box is drag-to-reposition.

`Application` (core) is the singleton orchestrator that wires inputs, simulation, playback, and the solver. Any change to `InputData` or the start position must retrigger `Application.runSimulation()`. **Don't break this wiring.**


## Where to find what

| Task | Look here |
| --- | --- |
| Orchestration / wiring | `core/.../Application.java` (read first); loader entry points `FabricParkourCalculator`, `Forge8ParkourCalculator`, `Forge12ParkourCalculator` |
| Input model & editing | `core/.../ui/InputData.java`, `InputRow.java`, `InputOverlay.java` |
| Save / load (JSON) | `core/.../save/SaveIO.java` (schema + parse), `SaveFile.java`, `FileSystemSaveStore.java`; mediator `core/.../SaveController.java` |
| UI shell / theming | `core/.../ui/MainWindowOverlay.java`, `OverlayManager.java`, `ui/theme/ThemeManager.java` (Catppuccin Mocha), `Settings.java` |
| Angle solver (core logic) | `core/.../anglesolver/AngleSolverEngine.java` (orchestrator), `AngleSolverState.java` |
| Solver inner loop | `core/.../anglesolver/solver/ExactJumpModel.java` (byte-exact X/Z stepper), `McSineTable.java`, `Constants.java` |
| Solver strategies | `solver/ClosedFormSolve.java` (fast convex), `SolveCore.java` (CMA-ES multistart), `LongRunSolver.java` (multi-jump), `BlockSolver.java` (obstacle avoidance) |
| Velocity finder | `core/.../anglesolver/velocity/VelocityFinder.java` (vx/vz sweep against a pad) |
| Solver UI | `core/.../ui/anglesolver/AngleSolverWindow.java`, `AngleSolverTable.java`, `SolverWidgets.java` |
| Constraint visualization | `core/.../render/ConstraintPlate.java`, `ConstraintShapes.java`; source `core/.../ui/anglesolver/AngleSolverConstraintSource.java` |
| Playback (TAS replay) | `core/.../PlaybackController.java`; loader `FabricPlaybackBridge` and Forge equivalents |
| Ports (core interfaces) | `core/.../ports/`: `MinecraftAccess`, `Simulator`, `BoxRenderer`, `PlaybackBridge`, `FilePickerPort` |
| Simulation (Fabric) | `loader-fabric-1.21.10/.../sim/SimulatorEntity.java`, `FabricSimulator.java`, `SimulatorInput.java` |
| Rendering (Fabric) | `loader-fabric-1.21.10/.../render/FabricWorldOverlayRenderer.java`, `FabricHudOverlayRenderer.java`; ImGui `imgui/ImGuiImpl.java` |
| Mixins (Fabric) | `loader-fabric-1.21.10/.../fabric/mixin/`; registered in `parkourcalculator.client.mixins.json` (Forge uses the FML event bus, no mixins) |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Leg0shii/ParkourCalculatorMod](https://github.com/Leg0shii/ParkourCalculatorMod) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
