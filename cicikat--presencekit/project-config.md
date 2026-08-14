---
trigger: always_on
description: > 每次开始任务前必读此文件。根据任务类型，再按需读对应的详细文档。
---

# AGENTS.md — Codex / Claude Code 工作入口

> 每次开始任务前必读此文件。根据任务类型，再按需读对应的详细文档。
> Codex 默认读取本文件。`CODEX.md` 是 `CLAUDE.md` 的兼容镜像，保留其协作偏好；若两者与本文件的当前工程约束冲突，以本文件和任务对应专题文档为准。

---

## 项目定位

角色（PresenceKit）是一个单用户 AI 陪伴系统，通过 QQ、桌宠和手机轮询通道与用户交互。

---

## 代码根目录

仓库克隆目录（即本文件所在目录）。

---

## 任务 → 读哪个文档

| 任务类型 | 必读文档 |
|---|---|
| 理解系统全貌、pipeline 流程 | `ARCHITECTURE.md` |
| 改记忆相关逻辑（episodic / user_identity / growth legacy / mood / event_log / fixation_pipeline / user_hidden_state） | `docs/memory.md` |
| 改 prompt 层结构、tag 规则、token 裁剪 | `docs/prompt-layers.md` |
| 改工具系统（新增工具、探针规则、桌面动作、execute() origin 闸门） | `docs/tools.md` |
| 改调度器（定时触发、主动消息） | `docs/scheduler.md` |
| 改 QQ / 桌宠通道、广播、WebSocket、跨通道接续 | `docs/channels.md`；桌面 v0.1 协议入口见 `docs/desktop-client-protocol.md` |
| 改多角色群聊、Stage session、共享 transcript、回合仲裁 | `docs/stage.md` |
| 改花园系统（情绪花槽、自动/被动浇水、采后处理、管理面板状态） | `docs/garden.md` |
| 理解事件/交互三维 envelope（realm/kind/lifecycle）、stimulus 边界、v0.1 约束 | `docs/interaction-event-model.md` |
| 修已知 bug / 查技术债 | `docs/known-issues.md` |
| 不确定设计意图、准入标准、禁止行为 | `DESIGN.md` |
| 改并发/锁/数据安全 | `docs/memory.md` → 七、并发保护 |
| 在 Codex / Claude Code Windows 环境运行测试、跨仓验证、处理沙箱报错 | `docs/dev-environment.md` |
| 改多模型接入、preset 路由、LLM provider 适配、prompt_style 转换 | `docs/model-presets.md` |
| 改鉴权/token/scope（`admin/auth.py`、`admin/scopes.py`、`admin/token_registry.py`） | `docs/security.md` |
| 改 ESP32 具身硬件固件（`firmware/presence-device/`） | `docs/presence-device-firmware.md`（协议/WS 通道侧见 `docs/channels.md`） |

---

## 关键文件速查

| 功能 | 文件 |
|---|---|
| 消息处理主流程 | `main.py` |
| Pipeline 四步骤 + tool loop（Brief 28 · Path C，`run_agentic_loop()`） | `core/pipeline.py` |
| Prompt 组装 | `core/prompt_builder.py` |
| Prompt 层级消融开关（对比/消融测试，只过滤注入不短路检索） | `core/prompt_ablation.py` |
| 话题标签规则 | `core/tag_rules.py` |
| 工具注册 + 调度 + 探针 | `core/tool_dispatcher.py` |
| Intiface / Buttplug 硬件控制 | `core/hardware/buttplug_client.py` / `core/hardware/device_registry.py` / `core/tools/hardware_tools.py` |
| 通道注册与广播 | `channels/registry.py` |
| 桌宠通道 WebSocket + 文件降级 | `channels/desktop_ws.py` / `channels/desktop.py`；协议权威指针 `docs/desktop-client-protocol.md` |
| 桌宠聊天 HTTP 入口 | `admin/routers/chat.py` → `/desktop/chat` |
| 手机通道 + 轮询接口 | `channels/mobile.py` / `admin/routers/mobile.py` |
| 统一 assistant turn sink | `core/turn_sink.py` |
| 多端 owner 对话串行锁 | `core/conversation_gate.py` |
| 多角色 Stage session / 共享 transcript / 回合仲裁 | `core/stage/models.py` / `core/stage/store.py` / `core/stage/arbiter.py` / `core/stage/runner.py` |
| 群聊梦境（Dream Stage，Brief 100：仅 sandbox、零回流、hard_exit 绝对） | `core/stage/dream_runtime.py`（`run_dream_stage_turn()`）/ `core/stage/dream_views.py`（`DreamStageCharacterView`）/ `core/stage/dream_state.py` + `dream_store.py` + `dream_settings.py`；端点 `admin/routers/group_dream.py`；详见 `docs/stage.md` §六 |
| 情景记忆 | `core/memory/episodic_memory.py` |
| 查询侧时间意图解析（Brief 48：解析"上周/前天/N天前"等，供 episodic/event_log/向量预取按时间窗过滤召回，纯规则无 LLM） | `core/memory/temporal_query.py` → `parse_query_time_range()`；接线点 `core/pipeline.py::fetch_context()` |
| 情景记忆淘汰暂存（遗忘=降级而非删除；上限裁剪批次存进 storyline_inbox，等周频聚合统一消费；原即时 digest 压缩已退役） | `core/memory/fixation_pipeline.py` → `handler_storyline_evicted_input()` |
| storyline 叙事弧层（append-only 存储 + 写API open_arc/append_node/set_arc_status；周频聚合 storyline_weekly；tagged 召回层 6h_storyline，Brief 80） | `core/memory/storyline.py` / `core/scheduler/triggers/storyline_weekly.py` |
| event_log 过期前抢救持久事实（age 27-29 天，产出走 important_facts 冲突裁决入口，不发言） | `core/scheduler/triggers/event_log_salvage.py` |
| 闲时整合 pass：episodic 存量近似重复合并（v1 零 LLM，复用写入时去重的同一相似度函数/阈值，核心记忆不参与，单轮上限10对）+ 向量库孤儿一致性核对（超阈值触发 rebuild） | `core/scheduler/triggers/memory_janitor.py` |
| 情绪状态 | `core/memory/mood_state.py` |
| 用户稳定行为模式 | `core/memory/user_identity.py` |
| 用户隐性状态 schema + primitives（Phase 3：apply_time_decay / reinforce_body_memory / consolidate_baselines 等已实现；source 类型守卫） | `core/memory/user_hidden_state.py` |
| 用户隐性状态 integrator（中期层 integrate_event/impression + Phase 3 长期层 integrate_body_cue*；TypeError 类型守卫；_assert_not_long_term；Brief 88：RealityEventType 扩至 5 类，get_trigger_counts() 观测计数） | `core/memory/user_hidden_state_integrator.py` |
| 用户隐性状态现实侧信号映射（Brief 88：对话侧五类事件判定 + body_memory cue 接线，挂 pipeline.post_process_slow） | `core/memory/user_hidden_state_reality_signals.py` |
| 用户隐性状态持久化（load / save 原子写入；load_dream_snapshot 只读 bucket 快照） | `core/memory/user_hidden_state_store.py` |
| 用户隐性状态衰减调度（12h decay tick + 7d consolidate tick，stamp_trigger，不发言；Brief 88：12h tick 内顺带 NO_INTERACTION 判定，逻辑日去重 stamp） | `core/scheduler/triggers/hidden_state_decay.py` |
| Dream snapshot 接入（Phase 4：tag-gated D4.5 只读注入；tag_gate helper；fail-closed） | `core/dream/dream_context.py` + `core/dream/dream_prompt.py` |
| Dream exit afterglow 回流接线（Phase 6：wire_afterglow_from_summary；tone 推导；fail-closed） | `core/dream/dream_exit_afterglow.py` |
| Reality prompt afterglow 软提示（Phase 7：_format_afterglow_soft_hint；只读；fail-closed；layer dream_afterglow_soft_hint） | `core/prompt_builder.py` → `_format_afterglow_soft_hint()` + `read_afterglow_residue()` |
| 调度器主循环 | `core/scheduler/loop.py` |
| 调度器状态机 / gating / proposer | `core/scheduler/state_machine.py` / `core/scheduler/gating.py` / `core/scheduler/proposer_registry.py` |
| 出梦主动开口触发器 | `core/scheduler/triggers/dream_exit.py` |
| 花园系统 | `core/garden/manager.py` / `core/garden/constants.py` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cicikat/PresenceKit](https://github.com/cicikat/PresenceKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
