---
trigger: always_on
description: An open-source plugin for Euphoria-like hit reactions in Godot 4.7+.
---

# Kickback — Physics-Based Reactive Characters for Godot 4.7+

## What this is

An open-source plugin for Euphoria-like hit reactions in Godot 4.7+.
Characters react dynamically to impacts using physics-driven ragdoll
and spring-based pose matching. Fully configurable and extensible.

**Engine**: Godot 4.7+ with Jolt physics.
**Language**: GDScript.

## Project structure

```
kickback/
├── CLAUDE.md
├── README.md
├── LICENSE
├── docs/
│   ├── STEP_BY_STEP.md              # Implementation history
│   ├── GODOT_CONSTRAINTS.md         # Engine quirks and workarounds
│   ├── REFERENCE.md                 # Technical reference: math, profiles, bone mapping
│   ├── INTEGRATION.md               # Integration guide: timing, layers, state machine, scoring
│   ├── FOOT_IK.md                   # Foot IK solver: planting, pelvis drop, anti-slide
│   ├── EUPHORIA_COMPARISON.md       # Feature-gap analysis vs Euphoria (honest scorecard)
│   ├── ROADMAP.md                   # Difficulty-weighted parity scorecard + milestones
│   ├── VERSIONING.md                # What the version numbers mean
│   └── SKELETON_MODIFIER_MIGRATION.md  # PhysicsRigSync → SkeletonModifier3D record
├── addons/
│   └── kickback/                    # The plugin (distributable)
│       ├── plugin.cfg
│       ├── kickback_plugin.gd       # Editor tool: "Add Kickback to Selected"
│       ├── kickback_character.gd    # Coordinator (detects mode, routes hits)
│       ├── kickback_manager.gd      # Global budget manager
│       ├── kickback_raycast.gd      # Hit detection utility (one-liner)
│       ├── kickback_layers.gd       # Collision-layer constants (active=4, demo godot-ragdoll=5)
│       ├── skeleton_detector.gd     # Auto-detect humanoid bones in any skeleton
│       ├── physics_rig_builder.gd   # Builds RigidBody3D ragdoll rig
│       ├── physics_rig_sync.gd      # Syncs physics → visible skeleton
│       ├── spring_resolver.gd       # Velocity-based spring pose matching
│       ├── foot_ik_solver.gd        # Two-bone foot IK (direct math → spring targets)
│       ├── active_ragdoll_controller.gd  # State machine (NORMAL/STAGGER/RAGDOLL/GETTING_UP/PERSISTENT)
│       ├── physics_collision_monitor.gd # Optional ragdoll-environment collision observer
│       ├── jolt_check.gd            # Jolt physics verification
│       ├── strength_debug_hud.gd    # F3 debug gizmos (auto-discovers all characters)
│       ├── editor/                  # Editor-only tooling
│       │   ├── kickback_inspector_plugin.gd
│       │   ├── kickback_status_panel.gd
│       │   ├── rig_baker.gd         # Bake persistent RigidBody3D + Joint nodes to scene
│       │   └── strip_root_motion.gd # Tool to strip root motion from animations
│       ├── icons/                   # Scene tree icons (SVG)
│       ├── presets/                  # Starter ImpactProfile .tres files
│       │   ├── bullet.tres, shotgun.tres, explosion.tres, melee.tres, arrow.tres
│       └── resources/               # Resource class definitions
│           ├── impact_profile.gd
│           ├── ragdoll_profile.gd
│           ├── ragdoll_tuning.gd
│           ├── bone_definition.gd
│           ├── joint_definition.gd
│           └── intermediate_bone_entry.gd
├── demo/                            # Demo scenes (not part of plugin) — 8 scenes
│   ├── partial_ragdoll_controller.gd # DEMO-ONLY: Godot PhysicalBoneSimulator3D ragdoll (the "built-in" side)
│   ├── hit_event.gd                 # DEMO-ONLY: hit data for partial_ragdoll_controller
│   ├── demo.tscn/gd                 # Kickback active ragdoll vs Godot's built-in PhysicalBoneSimulator3D
│   ├── shooting_range.tscn/gd       # FPS: 5 weapon profiles + ball-throw alt-fire (RMB)
│   ├── tuning_playground.tscn/gd    # Tuning Lab: 5 presets side-by-side + Custom char w/ live sliders
│   ├── signal_showcase.tscn/gd      # Visualizes all signals with floating popups
│   ├── stress_test.tscn/gd          # 20 characters, budget system, mass ragdoll
│   ├── animated_npc.tscn/gd         # Signal-driven NPC: patrol, hit, recover, resume
│   ├── foot_ik_demo.tscn/gd         # Foot IK on vs off over varied terrain
│   └── euphoria_showcase.tscn/gd   # All euphoria features: active resistance, sway, pain, injuries
├── assets/                          # Demo character (not part of plugin)
│   ├── characters/ybot/
│   └── animations/ybot/             # 21 animations (idle, walk, run, flinch, get-up, react, injured, kip-up)
└── project.godot
```

## Architecture

### Design principles
- **Physics controllers emit signals, don't play animations.** Animation handling is the user's responsibility. Connect to `stagger_started`, `recovery_started`, `recovery_finished`, `hit_absorbed`, `balance_changed`, `fatigue_changed`, `recovery_interrupted`, `pain_changed`, `threat_anticipated`, `region_injured` signals.
- **Animation-agnostic.** The physics core reads `Skeleton3D.get_bone_pose()` — works with AnimationPlayer, AnimationTree, or custom animation systems.
- **All configuration via Resources.** `RagdollProfile` (skeleton mapping) and `RagdollTuning` (physics feel) are assignable on `KickbackCharacter`. Null = auto-detected Mixamo defaults.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blugart-dev/kickback](https://github.com/blugart-dev/kickback) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
