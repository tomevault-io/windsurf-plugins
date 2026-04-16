---
trigger: always_on
description: A browser-playable 3D first-person training simulator for commercial duct cleaning technicians. Built for Carolina Quality Air. The goal: someone plays through this game thoroughly, shows up to a real job site, and fails at nothing.
---

# Duct Cleaning Simulator

## What This Is

A browser-playable 3D first-person training simulator for commercial duct cleaning technicians. Built for Carolina Quality Air. The goal: someone plays through this game thoroughly, shows up to a real job site, and fails at nothing.

## Tech Stack

- **Engine**: Babylon.js (latest v7+) — NOT Three.js. Babylon has built-in collision detection, gravity, FreeCamera (FPS-style WASD+mouse), physics plugins, action manager, and audio. Do not use Three.js.
- **Build**: Vite + vanilla TypeScript. No React, no framework. Babylon manages its own canvas and DOM.
- **Package manager**: npm
- **Deployment**: Static build → Cloudflare Pages
- **Style**: Industrial training aesthetic. Safety yellow (#FFD700) accents, dark grays (#1a1a1a, #2d2d2d), blueprint-style diagrams. Functional, utilitarian, not pretty. Think OSHA manual meets video game.

## Architecture

```
src/
  main.ts              # Entry point, engine init, scene loader
  scenes/
    MainMenu.ts        # Title screen, scenario select
    BuildingScene.ts   # The main 3D walkable environment
    DuctCrossSection.ts # 2D overlay for inside-duct cleaning view
  systems/
    PlayerController.ts # FPS movement, pointer lock, interaction raycasting
    GameState.ts       # Phase tracking, scoring, task completion
    EquipmentSystem.ts # Inventory, tool selection, equipment interaction
    HVACSystem.ts      # Duct network model, airflow direction, debris state
    ScoringSystem.ts   # Points, deductions, bonuses, final grade
    PatchingSystem.ts  # Access hole cutting, patching to code
    PressureWashSystem.ts # Grill cleaning sub-task
  ui/
    HUD.ts             # Babylon GUI overlay — task list, score, current tool
    EquipmentSelect.ts # Van/trailer loadout screen
    PhaseOverlay.ts    # Phase transition screens
    ScoreCard.ts       # End-of-job report
  models/
    BuildingGenerator.ts # Procedural commercial building floor
    DuctNetwork.ts     # Duct layout generation (trunk, branches, VAVs)
    Equipment.ts       # 3D representations of tools/equipment
  data/
    scenarios.ts       # Building configs, difficulty, system types
    equipment.ts       # Tool definitions, compatibility rules
    problems.ts        # Random event definitions
    scoring.ts         # Point values, deduction rules
  utils/
    constants.ts       # Colors, sizes, physics params
    audio.ts           # Sound effects manager
```

## Game Design

### Core Loop

Player receives a job ticket → loads equipment from van/trailer → enters building → assesses HVAC system → sets up (plastic, negative air, tubing) → cleans ducts (returns first, then supply) → patches access holes → pressure washes grills → cleans coils → reinstalls everything → cleanup → scored

### The 6 Phases

1. **Pre-Job**: Read job ticket. Select equipment loadout from van/trailer inventory. Vehicle check (random: low fuel, flat tire, check engine). Forgetting items = time penalty later.

2. **Arrival & Assessment**: Enter building. Find air handler. Identify system type (split, RTU, PTAC, VAV). Count registers (supply vs return). Plan access points. Lay plastic sheeting under work areas.

3. **Setup**: Connect tubing from negative air machine (squirrel cage) to trunk line. Run compressor hose for agitation wand. Quick-connect sections of 8-10" tubing (duct tape joints). Position portable vacuums at access points.

4. **Execution**: Clean returns first (upstream), then supply. Snake agitation wand with compressed air heads (forward/reverse). Cut access holes where needed (every 12ft or at turns). Use correct technique per material:
   - Rigid metal: aggressive agitation OK
   - Flex duct: gentle only (aggressive = collapse, -25 points)
   - Ductboard: air wash only (contact = fiber release, -20 points)
   
5. **Completion**: Patch all access holes to code:
   - 1" holes → pop plugs
   - 8" holes → sheet metal square patch, screws at corners and sides, mastic around seam, mastic on patch face, roll insulation back, seal with FSK tape (not duct tape)
   - Pressure wash all grills/registers (tarp down, garden hose to pressure washer, scrub stubborn ones)
   - Clean coils in air handler (coil cleaner spray + degreaser/Simple Green diluted)
   - Replace filters
   - Reinstall all registers/grills
   
6. **Cleanup**: Pull plastic sheeting. Sweep/dustpan debris. Pack equipment. Final walkthrough.

### HVAC System Model

The simulator must accurately model:

- **Air Handler**: Main unit that conditions air. Contains coils (evaporator), blower, filter rack.
- **Trunk Line**: Main duct leaving the air handler. Carries conditioned air.
- **Branch Ducts**: Smaller ducts splitting off trunk to individual zones/rooms.
- **VAV Boxes** (Variable Air Volume): Smart damper boxes between trunk and branch ducts. Control airflow per zone via thermostat. Have internal damper, possibly reheat coil. Player cleans these.
- **Supply Registers**: Where conditioned air enters rooms. Smaller, more numerous, may have directional louvers. Should be relatively clean unless mold/filter neglect.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bibijeebi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
