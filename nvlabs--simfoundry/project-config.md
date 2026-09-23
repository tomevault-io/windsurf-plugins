---
trigger: always_on
description: SPDX-FileCopyrightText: Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
---

<!--
SPDX-FileCopyrightText: Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
SPDX-License-Identifier: Apache-2.0
-->

# SimFoundry — agent guide

Orientation for coding agents. This covers what the code does not say out loud: which
environment to run in, which trees are safe to edit, and which contracts break silently.
For what the project *is*, read [README.md](README.md).

## What this repo does

SimFoundry turns a short real-world video into a physics-ready OmniGibson scene: segment
the objects, reconstruct geometry, generate textured meshes, compile a scene with physical
parameters and task proposals. The pipeline runs in three stages under
[`scripts/pipeline/`](scripts/pipeline/):

| Stage | Directory | Produces |
|---|---|---|
| A — reconstruction | `A_reconstruction/stages/` | Depth, segmentation, meshes, `s11_sim/scene_objects_info.json`, `s12_physics/pb_scene_poses.json`, an OmniGibson scene |
| B — augmentation | `B_augmentation/stages/` | Digital-cousin variants, scene sampling, task proposals |
| C — application | `C_application/stages/` | Teleop, demo generation, policy evaluation |

Scene editing sits between B and C: a human corrects what the pipeline got wrong — object
poses, scales, cameras — before the scene is used for learning or evaluation. Everything
downstream inherits those corrections.

## Environments

There are many conda envs on a dev box (`3dgrut`, `da3`, `hunyuan`, …); most belong to a
single pipeline stage. For general work:

- **`simfoundry`** — the main env. OmniGibson, Isaac Sim, torch. Used by the pipeline, the
  OmniGibson editor, and `settle.py`. `find_settle_python` in `light_editor/server.py`
  checks that the env's `omnigibson` resolves *inside this repo*.
- **`simfoundry-editor`** — the light editor's env, no GPU. Created by
  `scripts/installation/install_light_editor.sh`; re-run that after a pull. See the trap below.

> **Trap: `omnigibson` can resolve outside this checkout.** An env may carry this repo's
> `simfoundry` package but a *different* clone's OmniGibson. The editor then runs against
> the wrong `deps/` tree and stamps the wrong OmniGibson version into saved scenes.
> `run_editor.sh` warns about this at startup; do not ignore it. Check with
> `python -c 'import omnigibson; print(omnigibson.__file__)'`.

> **Trap: the light editor needs its own venv.** It depends on standalone `usd-core`, while
> Isaac Sim ships its own `pxr` that is only importable once Kit is running. Installing
> `usd-core` into `simfoundry` risks shadowing that copy. Never merge
> `light_editor/requirements.txt` into the conda envs.

## Do not edit `deps/`

`deps/BEHAVIOR-1K/` holds OmniGibson and is **not tracked by this repo** — `git ls-files deps/`
is empty. An edit there exists on one machine and is erased by the next `install.sh` run.

Anything requiring an OmniGibson change must become an upstream PR or a patch file in
[`patches/`](patches/) (the mechanism exists for 3dgrut, Hunyuan, FoundationPose). Keep work
in `simfoundry/` and `scripts/`.

The dependency is pinned by `scripts/installation/install_simfoundry.sh`, not by this repo's
git. A `git pull` inside `deps/BEHAVIOR-1K` can break tooling with no SimFoundry commit —
suspect the pin first when something breaks for one person and not another. The editors reach
into OmniGibson internals through its `lazy` passthrough, which is not a stable public API.

## The saved-scene contract

A scene state JSON (`<scene>_scene_state_<tag>.json`) is the interchange format between the
editors and the rest of the pipeline. Top-level keys: `versions`, `metadata`, `state`,
`init_info`, `objects_info`, `ground_plane_info`, plus optional `viewer_camera_state`,
`lighting_state`, `mesh_background_state`.

The editable quantities live in predictable places:

```
geometry   objects_info.init_info[name].args.usd_path            (relative to the JSON)
scale      objects_info.init_info[name].args.scale
pose       state.registry.object_registry[name].root_link.pos / .ori
friction   objects_info.init_info[name].args.link_physics_materials
                                       {<link>: {static_friction, dynamic_friction}}
mass       objects_info.init_info[name].args.mass                (kg)
joints     state.registry.object_registry[name].joint_pos         (the DOF array)
limits     objects_info.init_info[name].args.joint_limits
                                       {<joint>: {lower, upper}}
```

`friction` and `mass` are written by the light editor's physics panel and are
**not symmetric**. `link_physics_materials` is a real OmniGibson constructor kwarg
(`object_base.py`) and is applied to the link's collision meshes at load, so a
friction written here works everywhere with no consumer change. `mass` is
recorded only: OmniGibson has no load-time mass kwarg — `RigidPrim` reads one
from its load_config, but `EntityPrim` builds each link's load_config from a
fixed whitelist that mass is not on, and `BaseObject.mass` raises on assignment.
PhysX reads `physics:mass` from the USD, which rule 1 below forbids editing (and
which is shared across scenes anyway). **A consumer that wants to honour a
scene's mass applies it after load with `obj.root_link.mass = args["mass"]`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVlabs/SimFoundry](https://github.com/NVlabs/SimFoundry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
