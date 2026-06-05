---
trigger: always_on
description: Blender add-on that bridges Geometry Nodes geometry to GPU-accelerated physics simulation via NVIDIA Newton (built on Warp). Interactive: simulates live in the viewport on frame change.
---

# Warbler — Developer Notes for Claude Code

Blender add-on that bridges Geometry Nodes geometry to GPU-accelerated physics simulation via NVIDIA Newton (built on Warp). Interactive: simulates live in the viewport on frame change.

---

## Architecture

```
GN geometry (points / mesh / curves)
        ↓  GeometrySet  (geometryset.py)
   attributes dict
        ↓  SimulatorXPBD.build()  (simulation.py)
   newton.ModelBuilder
        ↓  .finalize()
   newton.Model + State × 2 + SolverXPBD
        ↓  frame_change_post handler  (manager.py)
   SimulatorXPBD.step()  every frame
        ↓
   Blender objects updated (rigid body transforms, particle pointcloud, cloth mesh)
```

### Key files

| File | Purpose |
|---|---|
| `warbler/__init__.py` | `register()` / `unregister()` — wires up classes and the frame handler |
| `warbler/simulation.py` | `SimulatorXPBD` — the main simulation class |
| `warbler/geometryset.py` | `GeometrySet` — evaluates GN output, reads attributes |
| `warbler/props.py` | All Blender properties (`SimulationListItem`, `WarblerObjectProperties`) |
| `warbler/manager.py` | `SimulationManager`, `get_manager()`, frame-change handler |
| `warbler/rigid.py` | `RigidObject` wrapper — reads shape config from `wb` properties |
| `warbler/panel.py` | `WB_PT_WarblerPanel` — 3D viewport N-panel |
| `warbler/ops.py` | Operators: Add/Remove/Compile simulation |
| `warbler/utils.py` | `get_scene()`, `wp_transform()`, `blender_rotation()` |
| `FEATURES.md` | Attribute roadmap — what GN attributes map to what Newton params |

---

## Simulation lifecycle

1. User clicks **Compile** → `WB_OT_CompileSimulation` → `SimulatorXPBD.compile()` → `build()` + `finalize()`
2. `build()` — creates `newton.ModelBuilder`, adds rigid bodies, particles, cloth
3. `finalize()` — calls `builder.finalize()`, creates two states, solver, contacts; creates Blender output objects (`ParticleObject`, `WarblerCloth`)
4. Every frame change → `_step_simulations(scene, depsgraph)` → `SimulatorXPBD.step()`
5. `step()` order: `_update_simulation_from_blender` → `simulate` (substep loop) → `_update_blender_from_simulation` → visualisation write-backs

---

## GeometrySet priority

`GeometrySet.data` checks `eval_obj.evaluated_geometry()` in order:
**pointcloud → curves → mesh** (GN output), then falls back to `eval_obj.data` (base object).

**Critical**: `_eval_geom` must be stored as an instance attribute. Data blocks from `evaluated_geometry()` are owned by the temporary GeometrySet and freed when it goes out of scope. Any access after losing the reference causes `ReferenceError: StructRNA of type Mesh has been removed`.

---

## Newton patterns

- `add_cloth_mesh(pos=wp.vec3(...), rot=wp.quat(...), vel=wp.vec3(...), ...)` — **must** use Warp types, not plain tuples. Plain tuples raise "Built-in functions cannot be called with non-Warp array types".
- `particle_grid` is `None` unless `particle_count > 1` AND `particle_max_radius > 0`. Never call `particle_grid.build()` — the solver does it internally.
- `contacts = model.contacts()` pre-allocate once in `finalize()`, pass to `model.collide(state, contacts)` and `solver.step(state_in, state_out, control, contacts, dt)`.
- `particle_q_init` is cloned from `state_in.particle_q` at the start of every `solver.step()`. Writing updated positions for kinematic (pinned) particles into `state_0.particle_q` before the step automatically propagates those positions as the kinematic target — no need to touch `particle_q_init` directly.
- `add_body(is_kinematic=True)` for user-controlled rigid bodies. Kinematic bodies collide with particles (velocity-driven impulses) but are never moved by physics forces. Previously these were fighting the per-frame position override.
- `builder.particle_mass` is a plain Python list before `finalize()` — you can zero out pinned cloth vertices by index: `builder.particle_mass[cloth_start + i] = 0.0`.

---

## GN attribute conventions

Warbler reads these named attributes from geometry at compile time:

**POINT domain (particles and cloth vertices)**

| Attribute | Type | Used for |
|---|---|---|
| `position` | `FLOAT_VECTOR` | Initial positions (required) |
| `velocity` | `FLOAT_VECTOR` | Initial velocities |
| `mass` | `FLOAT` | Per-particle mass |
| `radius` | `FLOAT` | Collision radius |
| `pinned` | `BOOLEAN` | `True` → kinematic (zero mass, position driven by GN each frame) |

For cloth, `pinned=True` zeros the vertex mass in `builder.particle_mass` during `_add_cloth()`. For particles, it sets `ParticleFlags` to 0 (not ACTIVE). Both are re-read from GN every frame so animated pin positions work.

**Cloth FACE / EDGE domain** — planned, not yet implemented. See `FEATURES.md`.

---

## Cloth simulation

- Source: any mesh object or GN-output mesh
- Vertices are transformed to world space before passing to `add_cloth_mesh()`
- Face indices are triangulated via `mesh.calc_loop_triangles()`
- Output: `WarblerCloth` Blender mesh object, updated every frame
- Particle range tracked by `_cloth_particle_start` / `_cloth_particle_count`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BradyAJohnston/warbler](https://github.com/BradyAJohnston/warbler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
