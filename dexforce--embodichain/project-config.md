---
trigger: always_on
description: **IMPORTANT**: The Python package name is `embodichain` (all lowercase, one word).
---

# EmbodiChain — Developer Reference

## Package Name

**IMPORTANT**: The Python package name is `embodichain` (all lowercase, one word).
- Repository folder: `EmbodiChain` (PascalCase)
- Python package: `embodichain` (lowercase)

## Project Structure

```
EmbodiChain/
├── embodichain/                  # Main Python package
│   ├── agents/                   # AI agents
│   │   ├── datasets/             # Datasets and data loaders for model training
│   │   ├── engine/               # Online Data Streaming Engine
│   │   ├── hierarchy/            # LLM-based hierarchical agents (task, code, validation)
│   │   ├── mllm/                 # Multimodal LLM prompt scaffolding
│   │   └── rl/                   # RL agents: PPO algo, rollout buffer, actor-critic models
│   ├── data/                     # Assets, datasets, constants, enums
│   ├── lab/                      # Simulation lab
│   │   ├── gym/                  # OpenAI Gym-compatible environments
│   │   │   ├── envs/             # BaseEnv, EmbodiedEnv
│   │   │   │   ├── managers/     # Observation, event, reward, record, dataset managers
│   │   │   │   │   └── randomization/  # Physics, geometry, spatial, visual randomizers
│   │   │   │   ├── tasks/        # Task implementations (tableware, RL, special)
│   │   │   │   ├── action_bank/  # Configurable action primitives
│   │   │   │   └── wrapper/      # Env wrappers (e.g. no_fail)
│   │   │   └── utils/            # Gym registration, misc helpers
│   │   ├── sim/                  # Simulation core
│   │   │   ├── objects/          # Robot, RigidObject, Articulation, Light, Gizmo, SoftObject
│   │   │   ├── sensors/          # Camera, StereoCamera, BaseSensor
│   │   │   ├── robots/           # Robot-specific configs and params (dexforce_w1, cobotmagic)
│   │   │   ├── planners/         # Motion planners (TOPPRA, motion generator)
│   │   │   └── solvers/          # IK solvers (SRS, OPW, pink, pinocchio, pytorch)
│   │   ├── devices/              # Real-device controllers
│   │   └── scripts/              # Entry-point scripts (run_env, run_agent)
│   ├── toolkits/                 # Standalone tools
│   │   ├── graspkit/pg_grasp/    # Parallel-gripper grasp sampling
│   │   └── urdf_assembly/        # URDF builder utilities
│   └── utils/                    # Shared utilities
│       ├── configclass.py        # @configclass decorator
│       ├── logger.py             # Project logger
│       ├── math/                 # Tensor math helpers
│       └── warp/kinematics/      # GPU kinematics via Warp
├── configs/                      # Agent configs and task prompts (text/YAML)
├── docs/                         # Sphinx documentation source + build
│   └── source/                   # .md doc pages (overview, quick_start, features, resources)
├── tests/                        # Test suite
├── .github/                      # CI workflows and issue/PR templates
├── setup.py                      # Package setup
└── VERSION                       # Package version file
```

---

## Code Style

### Formatting

- **Formatter**: `black==24.3.0` — run before every commit.
  ```bash
  black .
  ```

### File Headers

Every source file begins with the Apache 2.0 copyright header:

```python
# ----------------------------------------------------------------------------
# Copyright (c) 2021-2026 DexForce Technology Co., Ltd.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ----------------------------------------------------------------------------
```

### Type Annotations

- Use full type hints on all public APIs.
- Use `from __future__ import annotations` at the top of every file.
- Use `TYPE_CHECKING` guards for circular-import-safe imports.
- Prefer `A | B` over `Union[A, B]`.

### Configuration Pattern (`@configclass`)

All configuration objects use the `@configclass` decorator (similar to Isaac Lab's pattern):

```python
from embodichain.utils import configclass
from dataclasses import MISSING

@configclass
class MyManagerCfg:
    param_a: float = 1.0
    param_b: str = MISSING  # required — must be set by caller
```

### Functor / Manager Pattern

Managers (observation, event, reward, randomization) use a `Functor`/`FunctorCfg` pattern:

- **Function-style**: a plain function with signature `(env, env_ids, ...) -> None`.
- **Class-style**: a class inheriting `Functor`, with `__init__(cfg, env)` and `__call__(env, env_ids, ...)`.
- Registered in a manager config via `FunctorCfg(func=..., params={...})`.

```python
from embodichain.lab.gym.envs.managers import Functor, FunctorCfg

class my_randomizer(Functor):
    def __init__(self, cfg: FunctorCfg, env):
        super().__init__(cfg, env)

    def __call__(self, env, env_ids, my_param: float = 0.5):
        ...
```

### Docstrings


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DexForce/EmbodiChain](https://github.com/DexForce/EmbodiChain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
