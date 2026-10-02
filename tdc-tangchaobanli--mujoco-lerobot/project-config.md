---
trigger: always_on
description: 基于 MuJoCo 的 LeRobot 仿真环境：**数据采集 → 策略训练 → 评估**。全部 YAML 配置驱动、任务无关：任何新任务只需新增 teacher + 配置，不需要改动框架。
---

# AGENTS.md — mujoco-lerobot

基于 MuJoCo 的 LeRobot 仿真环境：**数据采集 → 策略训练 → 评估**。全部 YAML 配置驱动、任务无关：任何新任务只需新增 teacher + 配置，不需要改动框架。

## 设计原则（务必遵守）

1. **任务无关框架**：`configs/`（配置加载）、`simulate/`（MuJoCo 封装 / IK / 渲染）、`data/`（采集链路）均为通用组件，**不含任何特定任务的逻辑**——包括"如果任务是 X 就……"的分支。
2. **特定任务代码只允许放在 `src/mujoco_lerobot/mujoco_lerobot/data/teachers/`**：teacher（脚本式/遥操作，含任务专属状态机、成功判定、录制决策）+ 任务专属 YAML（`configs/tasks/`、`configs/scenes/`、`configs/teachers/`、`configs/dataset/`）。外部任务包也可自行注册 teacher。
3. **框架不满足新任务需求时，先对框架做兼容性变更**（通用化、可配置化），而不是在框架里加任务特化代码。变更须保持向后兼容（默认值与旧行为一致），并用测试固定。
4. **接入点靠注册表与配置连接，而非代码分支**：teacher 用 `@register_teacher("XxxTeacher")` 注册（含 `config_class`），任务用 `configs/tasks/tasks.yaml` 连接到 scene / teacher 配置；采集脚本、评估环境、训练链路均无任务分支。

### 新增任务的流程
1. **场景**：`assets/mujoco/scenes/<task>.xml` + `configs/scenes/scene_<task>.yaml`（机器人/相机/视角/`ik_solver`）。
2. **teacher**：在 `data/teachers/<task>_teacher.py` 继承 `Teacher`，`@register_teacher` 注册，实现 `step()`（输出各臂 `[x,y,z,qw,qx,qy,qz,gripper]` 目标位姿）与 `check_success()`（基于物理状态的评估判定）；遥操作类覆盖 `recording_decision()` 与 `start_collection()/end_collection()` 钩子。配 `configs/teachers/<task>.yaml`。
3. **连接**：`configs/tasks/tasks.yaml` 注册任务（scene_file + scene_config_file + teacher_config_file + domain_randomization）。
4. **采集/训练/评估** 直接用下方通用命令（`--task <task>`）。

## 架构 / 包布局

- `src/mujoco_lerobot/`（core 包 `mujoco_lerobot`）
  - `configs/`：YAML 配置加载（tasks / scenes / dataset / teachers / simulate）——`RobotConfig.ik_solver` 为 IK 参数
  - `simulate/`：`MujocoWrapper`（MuJoCo 封装）、`MinkIK`（mink IK）、`CameraRenderer`（多相机并行渲染，深度米制）——通用仿真组件
  - `data/`：采集链路（`ObservationCollector` / `SimulationManager` / `ScriptedTeacherController` / `KeyboardTeleopController` / `ResetManager` / `LeRobotDatasetWriter`）；**录制生命周期由 teacher 控制**（`run_episode` 每策略步询问 `controller.recording_decision`，返回 START/SAVED/DISCARDED/QUIT）；**采集会话钩子** `start_collection()/end_collection()` + `retry_limit`
  - `data/teachers/`：**任务专属代码唯一允许的位置**（pick_place / dual_pick_place / push_t + push_t 遥操作 `push_t_teleop.py`）
- `src/lerobot_env_mujoco_lerobot/`：LeRobot 评估环境插件（gym env，type=`mujoco_lerobot`；深度单位显式米）
- `src/lerobot_policy_Adaptive_ACT/`：自适应 ACT 策略插件（type=`adaptive_act`）
- `user/lerobot_policy/lerobot_policy_bimft/`：BiMFT 双臂多模态融合策略插件（type=`bimft`，适配多速率双臂数据集；模型代码内嵌、推理历史队列、相机角色可配置）
- `libs/mujoco_camrender/`：多相机并行渲染（C++ + pybind，git submodule）
- `configs/`：tasks / scenes / dataset / teachers / policy
- `scripts/collect_data.py`：一键采集（auto scripted teacher / keyboard / mouse / --append 追加）
- `tests/`：87 项测试（含无头遥操作测试）

## 端到端工作流

### 采集
```bash
# auto（scripted teacher）
uv run python scripts/collect_data.py --task-config configs/tasks/tasks.yaml \
    --task pick_place --dataset-config configs/dataset/dataset_pick_place.yaml \
    --episodes 50 [--no-render]

# PushT 鼠标遥操作（push_t 任务 teacher 模式下自动启动 pygame 2D 遥操作窗口，需 DISPLAY）
uv run python scripts/collect_data.py --task-config configs/tasks/tasks.yaml \
    --task push_t --dataset-config configs/dataset/dataset_push_t.yaml --episodes 20

# 追加采集（--episodes 为本轮新增条数；须配置一致性检查：fps/features/depth_unit/crf 硬性一致）
uv run python scripts/collect_data.py ... --append outputs/datasets/pick_place/<已有目录>
```
- 无头必须 `--no-render`（WSL/D3D12 下 `mujoco.viewer` 会 SIGSEGV）
- 输出 `outputs/datasets/<task>/<时间戳>/`（rgb+depth 视频编码，深度默认无损 HEVC）

#### PushT 鼠标遥操作（push_t 任务 teacher 模式自动启动）
- 按键：左键=切换鼠标控制 / 右键=开始·结束并保存 / 中键=丢弃当前集 / 滚轮=TCP 升降 / `[ ]`=鼠标灵敏度 / `- =`=滚轮灵敏度 / Esc=退出
- 窗口为 2D 俯视投影（pygame）：桌面范围、T 物块（绿，按 yaw 旋转）、目标 T（蓝）、当前 TCP（白点）、期望 TCP（黄十字）、状态栏（REC / MOUSE ON/OFF / TCP 高度 / PUSH OK-NO / 灵敏度）
- 配置：`configs/teachers/push_t.yaml`（window/tcp/push/sens/success）+ `scene_push_t.yaml` default_qpos（初始 TCP 落可推动带）+ `tasks.yaml`（t_obj_mount 随机化）
- 架构：`TeleopState`（线程安全，无 pygame 依赖，可无头测试）由窗口线程写、仿真线程读；teacher 经 `teacher_kwargs={"state": state}` 注入共享状态；`recording_decision` 每策略步先 `publish_teleop_state` 回写物理状态再消费鼠标事件；窗口生命周期由 `start_collection`/`end_collection` 钩子管理（`retry_limit=1`）

### 训练
```bash
uv run lerobot-train --config_path=configs/policy/adaptive_act.yaml \
    --dataset.root=outputs/datasets/pick_place/<时间戳> --steps=100000

# BiMFT 双臂策略（多速率数据集）
uv run lerobot-train --config_path=configs/policy/bimft.yaml \
    --dataset.root=outputs/datasets/dual_pick_place/<时间戳> --steps=100000
```
- 纯 RGB：`input_features` 只选 rgb；RGBD：rgb+depth 用 `concat_visual_features` 拼 4 通道（conv1 自动适配）
- 深度单位两端显式米（记录侧 features info + 训练侧 `dataset.depth_output_unit: m`），不依赖补丁

### 评估
```bash
uv run lerobot-eval --env.type=mujoco_lerobot --env.task=pick_place \
    --env.dataset_config=configs/dataset/dataset_pick_place.yaml \
    --policy.path=<实际训练 output_dir>/checkpoints/last/pretrained_model \
    --eval.n_episodes=5 --env.max_episode_steps=4000
```
- `--policy.path` 必须匹配实际训练的 `--output_dir`（路径错会报 `ParsingError: Couldn't instantiate class EvalPipelineConfig`）
- 给足 `max_episode_steps=4000`（约 40s）

### 测试
```bash
PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 uv run pytest tests/
```
（`test_depth_unit_explicitly_unified_as_m` 需 HF 网络，离线环境跳过）

## 关键配置

- `configs/tasks/tasks.yaml`：任务 → scene / teacher 配置连接（含 domain_randomization）
- `configs/scenes/scene_*.yaml`：机器人 / 相机 / 视角 / **`robot[].ik_solver`**（mink IK 参数，见下）
- `configs/dataset/dataset_*.yaml`：记录格式 + `state.camera.depth_range` + `depth_crf`/`rgb_crf`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TDC-TangChaoBanLi/mujoco_lerobot](https://github.com/TDC-TangChaoBanLi/mujoco_lerobot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
