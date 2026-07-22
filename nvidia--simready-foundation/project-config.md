---
trigger: always_on
description: This file is durable project context for AI assistants working on
---

# SimReady AI Context

This file is durable project context for AI assistants working on
`simready-foundation`. Read it before making changes to SimReady specs,
features, profiles, or validation.

## Core Mental Model

SimReady Foundations is a framework for defining what "Sim Ready" means for
OpenUSD assets. The main documentation and spec content lives under:

`nv_core/sr_specs/docs`

The framework is built from four connected concepts:

- **Requirements** are atomic, testable rules with stable IDs, such as
  `UN.006`, `VG.014`, `RB.001`, or `DJ.011`.
- **Capabilities** group related requirements by domain, such as Core,
  Visualization, Physics Bodies, Isaac Sim, Hierarchy, Non-Visual Sensors, and
  Semantic Labels.
- **Features** are runtime/use-case contracts. A feature is usually defined by
  what an asset must do in a runtime, such as being minimal/placeable/visual,
  graspable by a robotic gripper, physically simulated, articulated, or usable
  in Isaac Sim.
- **Profiles** are named, versioned bundles of features for asset classes or
  target environments, such as Sim Ready props, PhysX props, Isaac-ready props,
  or robot bodies.

Validation ties these together. Each feature should have validation tests or a
clear validation strategy that proves an asset actually implements the required
behavior.

## Workflow From Guides

`nv_core/sr_specs/docs/guides/guides.md` is the entrypoint for the practical
workflow. It links the four guide areas that should shape spec work:

- **Features** (`guides/features/features.md`): a feature is a versioned bundle
  of requirement IDs plus documentation. A feature change normally requires a
  markdown file, a JSON manifest, requirement links, samples where useful, a
  validation strategy, and an entry in `docs/features/features.md`.
- **Profiles** (`guides/profiles/profiles.md`): a profile is a named, versioned
  list of exact feature versions in `docs/profiles/profiles.toml`. Existing
  profile versions are immutable. To adopt new feature behavior, add a new
  profile version and update the related profile markdown/index docs.
- **Feature adapters** (`guides/feature_adapters/feature_adapters.md`): adapters
  mutate assets from one feature/profile contract to another. Create or update
  adapters only when the conversion requires USD data changes, and make sure
  every feature difference between source and target profiles has a direct
  adapter path.
- **Runtime testing** (`guides/runtime_testing/runtime_testing.md`): runtime
  tests provide behavioral proof beyond static validators. The pipeline is
  `batch_maker` to generate jobs, `job_runner` to execute them, and
  `report_generator` to create JSON/HTML reports. Prefer `workspace
  runtime_tests` inside Kit. Generated job JSON should not be hand-edited.

Feature expansion has a specific meaning in these guides: a technology-specific
feature can replace a base requirement when the technology supports a valid
pattern that the base rule would reject. In that case, the tech feature should
carry an explicit full requirement list with the replaced base requirement
removed and the technology-specific requirement added; profiles choose which
feature applies.

## First-Read Path for AI Agents

When an AI agent is new to this repository or returning after a context reset,
use this read order before changing specs, profiles, validators, or skills:

1. `AGENTS.md` - durable repo context and current workflow rules.
2. `nv_core/sr_specs/docs/guides/guides.md` - guide index.
3. `nv_core/sr_specs/docs/guides/features/features.md` - feature structure,
   versioning, dependencies, and feature expansion.
4. `nv_core/sr_specs/docs/guides/profiles/profiles.md` - profile structure,
   `profiles.toml`, profile versioning, and feature bundles.
5. `nv_core/sr_specs/docs/guides/feature_adapters/feature_adapters.md` - how
   assets are mutated between feature/profile contracts.
6. `nv_core/sr_specs/docs/guides/runtime_testing/runtime_testing.md` - runtime
   validation entrypoint and links to job runner details.
7. The specific profile, feature, requirement, and validator files touched by
   the task.

Treat `nv_core/sr_specs/docs/profiles/profiles.toml` as the machine-readable
profile source of truth. Profile markdown is an authoring guide and must stay
in sync with `profiles.toml`, but validators consume the TOML feature list.

## Prop-Robotics Profile Workflow

The prop robotics profiles are the main current workflow targets:

- `Prop-Robotics-Neutral`: OpenUSD-neutral prop assets for robotics pipelines.
  The base profile validates Core, Minimal, rigid-body physics, grasp physics,
  and materials. Single-rigid-body props satisfy prop physics through
  `FET003_BASE_NEUTRAL`; `FET004_BASE_NEUTRAL` is separate multibody work and
  should be validated only for props intentionally authored with multiple rigid
  bodies and joints.
- `Prop-Robotics-Physx`: PhysX prop assets. This profile uses PhysX rigid-body
  and multibody feature variants, including PhysX collider and joint behavior.
- `Prop-Robotics-Isaac`: Isaac Sim composition plus PhysX prop physics. This
  profile adds Isaac composition requirements on top of the prop physics and
  grasp expectations.

When reasoning about a prop profile, inspect these files together:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/simready-foundation](https://github.com/NVIDIA/simready-foundation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
