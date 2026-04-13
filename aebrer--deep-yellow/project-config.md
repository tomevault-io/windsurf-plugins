---
trigger: always_on
description: **Project**: DEEP YELLOW - Turn-based Roguelike in Godot 4.6
---

# CLAUDE.md - Deep Yellow

**Project**: DEEP YELLOW - Turn-based Roguelike in Godot 4.6
**Developer**: Drew Brereton (aebrer) - Python/generative art background, learning game dev

General working style, communication preferences, debugging process, and commit conventions
are in the global `~/CLAUDE.md`. This file covers Deep Yellow-specific context only.

---

## Environment: CachyOS Native Linux

- Godot 4.6, custom-built with Tracy profiler integration
- Tracy-enabled binary: `/home/drew/projects/godot-tracy/godot-4.6/bin/godot.linuxbsd.editor.x86_64`
- Tracy viewer: `/home/drew/projects/godot-tracy/tracy/profiler/build/tracy-profiler`
- Tracy csvexport: `/home/drew/projects/godot-tracy/tracy/csvexport/build/tracy-csvexport`
- User handles all visual/interactive testing (GUI, controller, gameplay)

**Performance review (part of PR workflow):**
- Capture Tracy trace on the PR branch and compare against master branch baseline
- Export with: `tracy-csvexport trace.tracy | sort -t',' -k5 -rn | head -30` (top by total %)
- CSV columns: name, src_file, src_line, total_ns, total_perc, counts, mean_ns, min_ns, max_ns, std_ns
- Identify and crush any introduced bottlenecks before merging

**Headless commands Claude can run:**
```bash
godot --headless --quit 2>&1              # Validate project loads, autoloads init, no script errors
godot --headless --script test_script.gd  # Script validation
godot --headless --import                 # Trigger import of new scripts and textures
```

Headless mode auto-selects "Start Game" in the start menu, so it loads into level -1 with full chunk generation and in-game context — useful for debugging beyond just script validation.

When ready for testing, say: "This is ready for you to test. When you run it, you should see [expected behavior]."

---

## Key Design Principles

**Controller-First, Input Parity is NON-NEGOTIABLE**
- All inputs have BOTH controller and keyboard+mouse mappings
- "Keyboard" means MOUSE MOVEMENT + keyboard, not just keyboard keys
- Test BOTH control schemes before considering a feature "done"
- Standard third-person controls: right stick OR mouse for camera rotation
- Don't reinvent camera controls — follow industry conventions (Fortnite/Gears of War)

**Turn-Based, Not Real-Time**
- NO `delta` time for gameplay logic (only for animations/polish)
- Pressure comes from resources/escalation, not timers
- Each action is discrete and turn-based

**Forward Indicator Movement**
- Green arrow shows 1 cell ahead in camera direction
- Rotate camera to aim, RT/LMB to move forward (with hold-to-repeat)
- DO NOT re-implement stick-based directional aiming — it was removed for better input parity

**Direct File Editing**
- Always edit Godot resource files (.tscn, .tres) directly when possible
- Don't make user manually import/export through Godot UI
- Godot files are text-based — take advantage of that

**Drew is learning game dev**
- Explain Godot-specific and game-dev-specific patterns when relevant
- Reference Python equivalents when helpful
- Don't assume knowledge of common game dev terminology

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                 RAW INPUT LAYER                      │
│  Controller / Keyboard → Godot Input Actions         │
└────────────────┬────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────┐
│            INPUTMANAGER (Autoload)                    │
│  Device abstraction, deadzone, analog → 8-dir grid   │
└────────────────┬────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────┐
│            STATE MACHINE LAYER                       │
│  Player → InputStateMachine → Current State          │
│  States: Idle (FPV/Tactical), PreTurn, Executing,    │
│          PostTurn, AutoExplore                        │
└────────────────┬────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────┐
│            ACTION LAYER (Command Pattern)             │
│  States create Actions → Actions validate & execute   │
│  Enables replays, AI, undo (future)                   │
└─────────────────────────────────────────────────────┘
```

**Why these patterns:**
- **InputManager (Autoload)**: Single source of truth for input, device abstraction, future replay/remapping
- **State Machine**: Turn-based game has distinct input modes; each state isolated, transitions via signals
- **Command Pattern**: Decouples intent from execution; AI can reuse same actions

---

## Control Mappings

**Do NOT add controls without explicit user request.**

Authoritative control list is the `CONTROL_MAPPINGS` const in `scripts/ui/settings_panel.gd` — read that for current mappings. That's the single source of truth; don't duplicate it here.

**Camera System:**
- Game loads in FPV (first-person view) by default
- C/SELECT toggles FPV ↔ Tactical (third-person overhead)
- FOV: 90° default, range 60°-110°
- Examination crosshair in FPV, move indicator (green arrow) in Tactical

---

## 3D World Y Coordinates (Empirically Confirmed)

GridMap cell_size is `Vector3(2.0, 1.0, 2.0)` but visual Y doesn't map 1:1 to cell units.

| Y Value | What's There |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aebrer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
