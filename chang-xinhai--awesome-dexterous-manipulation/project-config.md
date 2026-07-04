---
trigger: always_on
description: This repository is a curated **Awesome-Dexterous-Manipulation** list for the robotics ecosystem around dexterous hands, tactile sensing, hand-centric manipulation tasks, learning/control methods, benchmarks, datasets, and infrastructure.
---

# AGENTS.md

## Mission

This repository is a curated **Awesome-Dexterous-Manipulation** list for the robotics ecosystem around dexterous hands, tactile sensing, hand-centric manipulation tasks, learning/control methods, benchmarks, datasets, and infrastructure.

The goal is not to be a generic robot-manipulation bibliography. The goal is to organize the most relevant resources for:
- dexterous hands, multi-finger grippers, tactile sensors, and multimodal hardware
- hand-centric manipulation capabilities such as in-hand manipulation, non-prehensile manipulation, dynamic dexterity, bimanual dexterity, and deformable-object skills
- robot learning and control methods that materially advance dexterous manipulation
- data collection pipelines, teleoperation systems, simulation stacks, benchmarks, and datasets
- surveys, reviews, and taxonomy resources that help structure the field

## Single-file content policy

All curated content lives in `README.md`.

Do **not** create `contents/` pages for paper organization.
Do **not** split the awesome list into multiple markdown content files unless the user explicitly requests it.

Allowed root-level files:
- `README.md` - the canonical awesome list
- `AGENTS.md` - governance and maintenance rules
- `LICENSE`
- `.gitignore`
- optional static assets under `assets/` or `imgs/` when they improve presentation

## Canonical section layout

`README.md` should use this top-level structure:
1. `Hardware & Sensor Systems`
2. `Dexterity Capabilities & Tasks`
3. `Methodology`
4. `Infrastructure`
5. `Surveys & Reviews`

Use `About`, `Must Read`, `News`, and `Contents` near the top of the README. Keep subsection anchors stable.

## Section definitions

### Hardware & Sensor Systems

Include resources about the physical interface layer for dexterous manipulation:
- dexterous hands and anthropomorphic hand platforms
- non-anthropomorphic hands and multi-finger grippers
- tactile sensors, robotic skin, force sensing, and visuo-tactile fingertips
- multimodal setups combining vision, touch, proprioception, force/torque, audio, or wearable capture

### Dexterity Capabilities & Tasks

Include work primarily defined by the manipulation capability or task family:
- prehensile manipulation: in-hand reorientation, rotation, finger gaiting, regrasping, tool use, extrinsic dexterity
- non-prehensile manipulation: pushing, sliding, flipping, rolling, pivoting
- dynamic and agile manipulation: catching, throwing, juggling, pen spinning, high-frequency object motion
- complex and specialized tasks: bimanual dexterous manipulation and deformable object manipulation

Do **not** turn this section into a broad manipulation task dump. The hand-centric dexterity connection should be clear.

### Methodology

Include learning, planning, control, and data-pipeline methods that are important for dexterous manipulation:
- reinforcement learning and sim-to-real
- imitation learning, behavior cloning, action chunking, and diffusion policies
- VLA/foundation models used for dexterous or fine-grained manipulation
- trajectory optimization, MPC, kinematics, and motion planning
- teleoperation, glove/exoskeleton capture, robot-free teaching, passive video, and internet data extraction

### Infrastructure

Include reusable infrastructure:
- simulators and physics engines
- benchmark suites and datasets
- robot descriptions, assets, environments, data formats, and training/evaluation stacks

### Surveys & Reviews

Include survey papers, tutorials, taxonomies, and review-style resources that help readers understand dexterous manipulation, tactile sensing, grasping, or robot manipulation more broadly.

## Inclusion policy

The repository uses **core + strong adjacent** scope.

### Include first
- canonical dexterous manipulation papers and projects
- widely used dexterous hands, tactile sensors, and open hardware/software platforms
- datasets and benchmarks designed for hand-centric grasping, dexterous manipulation, tactile manipulation, or dexterous teleoperation
- methods demonstrated on dexterous hands, in-hand manipulation, bimanual dexterity, visuo-tactile manipulation, or contact-rich fine manipulation

### Include selectively
- general robot-learning policies when they are commonly used as dexterous manipulation baselines or infrastructure
- broad manipulation benchmarks when they include important dexterous-hand, tactile, deformable, or fine manipulation components
- humanoid manipulation work when dexterous-hand manipulation is a central contribution

### Usually exclude
- generic pick-and-place manipulation with weak dexterity relevance
- broad robot planning/control work that does not materially help dexterous manipulation
- unrelated industrial gripper resources without a clear dexterous or multi-finger angle
- unverified papers, rumor-like projects, or entries without a reliable public source

## Source priority

When adding or updating entries, use sources in this order:
1. official project page, official code repository, or official dataset page
2. arXiv, conference, journal, OpenReview, or proceedings page

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chang-xinhai/Awesome-Dexterous-Manipulation](https://github.com/chang-xinhai/Awesome-Dexterous-Manipulation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
