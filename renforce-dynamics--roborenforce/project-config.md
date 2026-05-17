---
trigger: always_on
description: This file contains persistent instructions for Claude Code when working in this repository.
---

# Claude Code Instructions for RoboRenforce

This file contains persistent instructions for Claude Code when working in this repository.

---

## Project Context

RoboRenforce is a robot learning framework combining:
1. **Current state**: Pure RL framework (PPO, SAC, DSAC, GAIL, etc.)
2. **Goal**: Extend to support VLA pretraining + RL fine-tuning

**Key files to review before major changes**:
- `TODO.md` - Task breakdown for VLA integration
- `.claude/project-overview.md` - Architecture and design decisions
- Memory system (auto-loaded) - Reference implementations and patterns

---

## Code Standards

### 1. Always Use Configclass Pattern

All new components MUST follow the `@configclass` pattern:

```python
from RoboRenForce.utils.configclass import configclass, ModuleBaseCfg, MISSING

@configclass
class MyComponentCfg(ModuleBaseCfg):
    class_type: type[MyComponent] = MyComponent
    required_field: int = MISSING  # Forces user to provide
    optional_field: float = 1.0
    nested_cfg: OtherCfg = OtherCfg()  # Auto-wrapped in default_factory
    
class MyComponent(ModuleBase):
    def __init__(self, cfg: MyComponentCfg, *args, **kwargs):
        super().__init__(cfg)
        # Implementation...
```

**Why**: Ensures type safety, serializability, and consistency with existing codebase.

**Validation**: All configs must pass `.validate()` before use. Check for MISSING values.

### 2. Preserve Existing Architecture

**DO NOT** refactor the Runner → Algorithm → Components hierarchy unless explicitly requested.

**DO** extend it:
- New policy types → Add to `ActorCriticPack`
- New algorithms → Subclass `AlgorithmBaseCfg`
- New runners → Subclass `BaseRunnerCfg`

**Example**: VLA actor fits into existing `ActorCriticPackCfg(actor_cfg=VLAActorCfg(...))`

### 3. Reference Implementation Patterns

When implementing from references:

**LeRobot** (`.references/lerobot`):
- **Adopt**: Data format (Parquet + metadata), processor pipeline architecture
- **Adapt**: Use `@configclass` instead of `draccus` for configs
- **Cite**: Add docstring comment referencing source file

**Psi0** (`.references/Psi0`):
- **Adopt**: VLA model architecture (VLM + action head), mixed-dataset training
- **Adapt**: Use `ModuleBaseCfg` instead of Pydantic `BaseModel`
- **Cite**: Reference source file in docstrings

**RoboTwin** (`.references/RoboTwin`):
- **Adopt**: LoRA fine-tuning patterns, environment integration
- **Adapt**: Integrate with RoboRenforce's runner framework
- **Cite**: Reference source file in docstrings

**Template for citing references**:
```python
class VLAActor(ModuleBase):
    """
    VLA actor combining VLM backbone and action head.
    
    Reference: .references/Psi0/src/psi/models/psi0.py
    
    Adaptations:
    - Uses RoboRenforce configclass instead of Pydantic
    - Integrates with ActorCriticPack framework
    """
```

### 4. File Organization

**New VLA components go in**:
- Models: `source/RoboRenForce/networks/vlm/`
- Actors: `source/RoboRenForce/components/actor/vla_actors.py`
- Runners: `source/RoboRenForce/runners/vla_pretrain_runner.py`, `vla_rl_runner.py`
- Data: `source/RoboRenForce/dataset/lerobot_dataset.py`
- Processors: `source/RoboRenForce/utils/processor/`
- Losses: `source/RoboRenForce/algorithms/losses/vla_losses.py`

**Scripts**:
- Training: `scripts/renforce/train_vla_pretrain.py`, `train_vla_rl_finetune.py`
- Data conversion: `scripts/data/rlds_to_lerobot.py`, `isaaclab_to_lerobot.py`

**Examples**:
- Configs: `source/demo_tasks/vla_pretrain/`, `source/demo_tasks/vla_finetune/`

### 5. Testing Requirements

Every new component needs:

1. **Unit test**: Test component in isolation
   ```python
   # tests/components/actor/test_vla_actor.py
   def test_vla_actor_forward():
       cfg = VLAActorCfg(...)
       actor = cfg.construct_from_cfg(dim_params={...})
       output = actor(obs)
       assert output.shape == expected_shape
   ```

2. **Config test**: Serialize/deserialize
   ```python
   def test_vla_actor_cfg_serialization():
       cfg = VLAActorCfg(...)
       cfg_dict = cfg.to_dict()
       cfg_restored = VLAActorCfg()
       cfg_restored.from_dict(cfg_dict)
       assert cfg == cfg_restored
   ```

3. **Integration test**: Test in full pipeline (if applicable)
   ```python
   def test_vla_pretrain_runner():
       runner = VLAPretrainRunner(cfg, env, ...)
       runner.learn(num_iterations=10)  # Short run
       assert runner.logger.has_logged("action_loss")
   ```

**Run tests before committing**: `pytest tests/`

---

## Common Tasks

### Adding a New VLA Model

1. **Create model class** in `source/RoboRenForce/networks/vlm/`:
   ```python
   @configclass
   class MyVLMCfg(ModuleBaseCfg):
       class_type: type[MyVLM] = MyVLM
       model_name: str = "my-vlm-model"
       # ... config fields
   
   class MyVLM(ModuleBase):
       def __init__(self, cfg: MyVLMCfg):
           # Load from HuggingFace, etc.
   ```

2. **Create action head config** in `source/RoboRenForce/components/actor/`:
   ```python
   @configclass
   class MyActionHeadCfg(ModuleBaseCfg):
       # ... config fields
   ```

3. **Create VLA actor config** combining them:
   ```python
   @configclass
   class MyVLAActorCfg(ModuleBaseCfg):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Renforce-Dynamics/RoboRenForce](https://github.com/Renforce-Dynamics/RoboRenForce) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
