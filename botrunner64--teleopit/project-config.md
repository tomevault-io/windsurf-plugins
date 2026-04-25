---
trigger: always_on
description: Teleopit is a lightweight, extensible, self-contained humanoid robot whole-body teleoperation framework. It integrates GMR (General Motion Retargeting) and supports train_mimic-exported ONNX RL policy inference.
---

# AGENTS.md

## Project Overview

Teleopit is a lightweight, extensible, self-contained humanoid robot whole-body teleoperation framework. It integrates GMR (General Motion Retargeting) and supports train_mimic-exported ONNX RL policy inference.

Language: Python 3.10+
Package: `teleopit` (installed via `pip install -e .`)
Config: Hydra/OmegaConf YAML files in `teleopit/configs/`

## Architecture

```
InputProvider (BVH file / Pico4 VR) → Retargeter (GMR) → ObservationBuilder (166D) → Controller (dual-input TemporalCNN ONNX) → Robot (MuJoCo + PD / Unitree SDK)
```

Module-internal isolation: all modules run in-process and communicate via `InProcessBus` (zero-copy). Core interfaces are defined as `typing.Protocol` in `teleopit/interfaces.py`.

## Supported Surface

- Training task: `General-Tracking-G1`
- Inference observation: `velcmd_history` (166D, dual-input ONNX with `obs` + `obs_history`)
- TemporalCNN actor/critic with larger dims (1024,512,256,256,128)
- Realtime inference uses a retargeted-reference timeline before observation build; `reference_steps=[0]` is the default production path

## Directory Structure

```
teleopit/                 # Core inference package
├── interfaces.py         # Protocol definitions: Robot, Controller, InputProvider, Retargeter, etc.
├── pipeline.py           # TeleopPipeline — thin sim runtime facade
├── runtime/              # Shared runtime assembly: config/path resolution, factories, CLI helpers
├── bus/                  # InProcessBus message pub/sub
├── configs/              # Hydra YAML configs
│   ├── default.yaml      # Offline sim2sim
│   ├── sim2real.yaml     # sim2real
│   ├── robot/g1.yaml     # G1 robot: XML path, PD gains, default angles, action dims
│   ├── controller/rl_policy.yaml
│   ├── input/bvh.yaml    # Offline BVH file input
│   └── input/pico4.yaml  # Pico4 realtime input
├── controllers/
│   ├── rl_policy.py      # RLPolicyController — single-input or dual-input ONNX inference with fail-fast dim checks
│   └── observation.py    # VelCmdObservationBuilder
├── inputs/
│   ├── bvh_provider.py       # BVHInputProvider — offline BVH file
│   ├── pico4_provider.py     # Pico4InputProvider — xrobotoolkit_sdk realtime body tracking input
│   ├── rot_utils.py          # Quaternion helpers for input-space transforms
│   └── zmq_provider.py       # ZMQInputProvider — onboard Pico4 realtime receiver
├── retargeting/
│   ├── core.py           # RetargetingModule + extract_mimic_obs()
│   └── gmr/              # Self-contained GMR code; heavyweight assets are downloaded into an ignored path
├── robots/
│   └── mujoco_robot.py   # MuJoCoRobot — MuJoCo sim wrapper
├── sim/
│   └── loop.py           # SimulationLoop — PD control at 1000Hz, policy at 50Hz
└── recording/            # HDF5Recorder
scripts/
├── run_sim.py            # Offline sim2sim pipeline
├── run_sim2real.py       # G1 sim2real control; supports offline BVH playback and Pico4
├── render_sim.py         # Render single BVH → 3 MuJoCo videos (mocap input, retarget, sim2sim)
├── compute_ik_offsets.py # Compute IK quaternion offsets for new BVH formats
└── setup_pico4.sh        # Pico4 environment setup helper
train_mimic/              # Training package
├── app.py                # Shared app helpers for train/play/benchmark
├── tasks/tracking/config/
│   ├── constants.py      # Public task constants
│   ├── registry.py       # Registers General-Tracking-G1 task
│   ├── env.py            # General-Tracking-G1 env builder
│   └── rl.py             # TemporalCNN PPO cfg
├── tasks/tracking/rl/
│   ├── runner.py         # ONNX export wrapper for policy + motion labels
│   ├── conv1d_encoder.py # 1-D CNN encoder for temporal history groups
│   └── temporal_cnn_model.py # TemporalCNN actor/critic model
└── scripts/
    ├── train.py          # Training entry point
    ├── play.py           # Checkpoint playback
    ├── benchmark.py      # Policy evaluation with tracking errors
    └── save_onnx.py      # Export TemporalCNN ONNX
```

## Key Technical Details

### Sim2Sim Pipeline
- Policy runs at 50Hz, PD control at 1000Hz (`decimation=20`, `sim_dt=0.001`)
- Action flow: `compute_action()` returns raw action → `get_target_dof_pos()` applies clip `[-10, 10]`, scale, and `default_dof_pos`
- Must use `g1_mjlab.xml` for sim2sim; `g1_mocap_29dof.xml` clamps torques to `±1 Nm` and is only for kinematic retarget visualization

### Multi-Viewer Support
`SimulationLoop` supports three simultaneous viewer windows controlled by the `viewers` config:

```bash
python scripts/run/run_sim.py controller.policy_path=policy.onnx viewers=sim2sim
python scripts/run/run_sim.py controller.policy_path=policy.onnx 'viewers=[mocap,retarget,sim2sim]'
python scripts/run/run_sim.py controller.policy_path=policy.onnx viewers=all
python scripts/run/run_sim.py controller.policy_path=policy.onnx 'viewers=[retarget,sim2sim]'
python scripts/run/run_sim.py controller.policy_path=policy.onnx viewers=none
```

- `sim2sim`: MuJoCo physics result
- `retarget`: kinematic retarget result
- `mocap`: retargeting input skeleton rendered by MuJoCo custom geoms

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BotRunner64/Teleopit](https://github.com/BotRunner64/Teleopit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
