---
trigger: always_on
description: **当前阶段**: 稳定回退点已建立（`ea204b1`），进入 v2.5：`runtime shadow bridge v1` 收口 + 动态 Pose Takeover 前置阶段
---

# Agents.md - 项目进度与交接文档

## 📅 项目当前状态 (Current Status)

**最后更新**: 2026-04-05
**当前阶段**: 稳定回退点已建立（`ea204b1`），进入 v2.5：`runtime shadow bridge v1` 收口 + 动态 Pose Takeover 前置阶段

### 🔖 当前稳定回退点（2026-04-05）
1. **稳定提交点**：`ea204b1`（`checkpoint runtime shadow bridge and dynamic unit fallback fixes`）。
2. **当前策略**：
   - 飞行单位、动态 `CUnit`、蒙皮单位已强制退出 `persistent cache`，避免阴影被静态缓存后停在原地；
   - `runtime shadow bridge v1` 与“对象身份直传桥”已落地，但仍以**只读桥接 + fallback 正确性优先**为主；
   - 动态 Pose Takeover 尚未正式点亮，当前目标是先稳定生命周期、崩溃追踪和接入时机。
3. **当前主阻塞**：
   - AutoTest 进图判定链与真实 in-map 验证仍需继续稳固；
   - 动态姿态接管的生产级接入点需要收敛到更安全的 `CSpriteUber_PreRenderAndUpdatePosePalette` 返回点；
   - 必须优先消除未处理异常与悬空指针问题，才能继续推进动态阴影主路径。

### 🎯 当前阶段目标（2026-04-05）
1. 在保持当前“动态单位不再被错误缓存”的前提下，继续推进 `runtimeModel + pose palette` 的安全接入。
2. 将动态阴影主路径从 `draw-time fallback freeze` 迁移到“静态模型资源 + 每帧姿态更新”。
3. 保留研究资料、桥接模块和崩溃证据链，确保后续可以安全回退与复盘。
4. 在进入下一轮性能优化前，先完成崩溃隔离、接入时机收敛与 AutoTest 稳定化。

### 🎯 本阶段目标（新增）
1. 在不牺牲当前功能与性能的前提下，完成架构解耦与模块化重排。
2. 将 `d3d9_war3_hook.cpp` 从“功能承载入口”降级为“编排入口”。
3. 建立统一 Hook 安装框架（地址解析、安装、降级、统计、日志）。
4. 建立可回归性能护栏，确保每个阶段重构后可量化验证“不倒退”。

### 🏗️ 项目结构总览（行业化 v1）
| 层级 | 目录 | 关键文件 | 职责 | 扩展入口 |
|---|---|---|---|---|
| Runtime / Bootstrap | `src/d3d9/war3/platform/` | `war3_runtime_bootstrap.*`, `war3_module_api.*` | 运行时初始化、模块生命周期、状态统计 | 在 `war3_module_api` 注册新模块 |
| Hook Orchestrator | `src/d3d9/war3/hooks/` | `war3_hook_address_book.*`, `war3_hook_install_util.*`, `war3_hook_*.cpp` | 地址解析、MinHook 安装、分域 Hook 编排 | 新增域时按 `War3HookXxx::Install` 接入 |
| Render Frontend | `src/d3d9/war3/render/` | `war3_scene_collector.*`, `war3_render_exec_batch.*`, `war3_render_queue_tracker.*` | 对象收集、批次桥接、队列追踪 | 在 collector/exec_batch 增强分类或桥接 |
| Frame / Pipeline | `src/d3d9/` + `src/d3d9/war3/render/` | `d3d9_war3_pipeline.*`, `war3_frame_graph.*` | BeforeUi/BeforePresent 编排与 pass 调度 | 在 `war3_frame_graph` 增减事件序列 |
| Feature Modules | `src/d3d9/` | `d3d9_war3_shadow*.cpp`, `d3d9_war3_ssao.cpp`, `d3d9_war3_aa.cpp` | 阴影/描边/SSAO/AA 等效果 | 按模块文件独立演进，避免回灌主入口 |
| Shader / Material | `src/d3d9/war3/shader/` + `src/d3d9/` | `war3_shaderpack.cpp`, `war3_shader_api.*` | ShaderPack、uniform 与材质覆盖 | 新增 pass 时先声明 API 再接管线 |
| Diagnostics / Perf | `src/d3d9/war3/tools/` | `war3_perf_monitor.*`, `war3_diagnostics_hub.*` | 性能采样、健康日志、HTML 报告 | 统一在 PerfMonitor 增指标，避免分散口径 |

### 🚀 使用指南（开发/验证/性能）
1. **编译**：`ninja -C build32`（必须通过后再进入下一阶段）。
2. **运行时日志**：DebugView 观察 `DXVK War3Hook`, `DXVK War3Diag`, `DXVK War3Shadow` 前缀。
3. **性能记录**：
   - 在 ImGui 面板启动/停止性能记录（停止时自动导出报告）。
   - 报告路径：`WarVK/Log/war3_perf_report_YYYY_MM_DD_HH_MM_SS.html`。
4. **性能窗口与缓存配置（可选环境变量）**：
   - `DXVK_WAR3_PERF_WINDOW_SEC`：报告统计窗口秒数（默认 1200）。
   - `DXVK_WAR3_PERF_HISTORY_SEC` / `DXVK_WAR3_PERF_HISTORY_FRAMES`：帧历史容量。
   - `DXVK_WAR3_PERF_PENDING_MAX`：GPU query 待处理上限。
5. **新增功能接入流程**：
   - 先在 `hooks` 中定义域入口；
   - 再在 `render/pipeline` 接事件；
   - 最后在 `tools` 补监控指标与回归口径。
6. **验收口径（当前强制）**：
   - 功能不回退（阴影/描边/JASS 时间链路稳定）；
   - `ninja -C build32` 通过；
   - 性能报告具备 `Avg/P95/P99 + Coverage + Untracked + Self/Inclusive` 四类指标。

### 🧱 行业化重构计划表（2026-02-22 起）
| 阶段 | 目标 | 主要工作 | 验收标准 | 预计时间 |
|---|---|---|---|---|
| P0 基线护栏 | 建立“可回归”底座 | 固化 benchmark 场景、日志采样、关键性能门限；整理功能回归清单 | `ninja -C build32` 稳定通过；可输出同场景对比报告 | 1-2 天 |
| P1 Hook 架构统一 | 消除重复安装与分散入口 | 新增 Hook AddressBook/Registry/Gate；主入口统一注册路径 | `d3d9_war3_hook.cpp` 仅保留编排；安装成功率/失败原因可观测 | 3-5 天 |
| P2 域迁移落地 | Render/Jass/Lifecycle/UI/Shadow 全域模块化 | 将域内 Hook 实现迁移到 `war3/hooks/*`；删除重复实现 | 不再存在同功能双实现；功能回归通过 | 4-6 天 |
| P3 桥接契约化 | 稳定渲染层与逻辑层边界 | 统一 `sceneNode/jHandle/unit/rawcode` 契约；补齐桥接断言与统计 | 描边/阴影匹配链路可解释、可追踪、可回归 | 5-7 天 |
| P4 设备热路径解耦 | 降低 `d3d9_device.cpp` 耦合度 | 抽离 ShadowCapture/Outline/BeforeUi 编排模块 | 热路径行为一致；CPU 指标不回退 | 7-10 天 |
| P5 配置与诊断标准化 | 降低开关复杂度 | 分层配置（dev/profile/release）；统一诊断输出 | `war3_internal_test_config.h` 收敛；诊断项可分级控制 | 3-5 天 |
| P6 文档行业化 | 形成可维护工程文档体系 | 更新 `docs/war3_shader_docs` 与研究文档结构图/模块说明 | 新成员可按文档完成定位与扩展 | 持续并行 |

### ✅ P0 当前落地状态（2026-02-22）
1. 已完成：全项目结构盘点与耦合点识别（Hook 重复实现、状态层分裂、热路径集中）。
2. 已完成：编译基线验证，`ninja -C build32` 通过（存在既有 warning，无阻塞错误）。
3. 已完成：`AGENTS.md` 与行业化看板同步到“v1 收官版本”，后续按 v2 计划推进。

### ✅ 已完成工作 (Completed)
1. **JASS 时间获取修复**: 
   - 修复了 `GetTimeOfDay` 无法获取时间的问题。
   - 重构了初始化时序：`ActivateWar3Runtime` -> `Hook_ExecuteJassFunction` -> `NET_EVENT_GAME_READY`。
   - 解决了早期 JASS Native 调用时的运行态同步问题（该桥接实现现已移除，保留原生运行时链路）。
   - 恢复了动态光影随时间变化的功能。
2. **基础解耦 (Hook Decoupling)**:
   - `NetEventHook` 已独立。
   - `ShaderManager` 初步建立。
   - 早期曾实现 JAPI 封装层；当前版本已移除相关桥接源码。
   - `Hook_WorldObjects_RenderGroup` 逻辑已抽取至 `RenderQueueTracker`，移除了 RenderQueue 指针操作的大量 hack 代码。
3. **性能与稳定性修复**:
   - **卡顿解决**: `ShaderManager` 实现懒加载 (Deferred Compilation)，消除 `ActivateWar3Runtime` 时的 I/O 卡顿。
   - **崩溃解决**: `ResetWar3RuntimeState` 优化了析构顺序，并且将核心单例 (`War3Renderer`, `RenderQueueTracker`, `ShaderManager` 等) 改为 **Leaky Singleton** 模式，彻底避免 Process Detach 时的静态析构顺序崩溃。
4. **性能诊断**:
   - 在 `ActivateWar3Runtime` 中添加了微秒级耗时统计日志 (`DXVK War3Hook Init Profile`)，用于定位启动卡顿的精确位置。
5. **三方向专项推进（ASM 驱动）**:
   - 新建统一研究目录：`docs/research/war3_render_issues/`，包含三个方向的独立文档。
   - **方向1（合批）**：在 `war3_render_queue.h` 收紧起批条件，新增 next `sceneNode` 可读性检查，减少 singleton 空转；并将 `FlushGroup` scope 移入 `pendingCount>1` 分支。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CallDisaster/War3VK](https://github.com/CallDisaster/War3VK) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
