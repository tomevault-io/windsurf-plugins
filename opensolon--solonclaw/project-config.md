---
trigger: always_on
description: 本仓库的基础架构思路学习和参考了开源项目 [HKUDS/nanobot](https://github.com/HKUDS/nanobot)。
---

# AGENTS

## 置顶说明

本仓库的基础架构思路学习和参考了开源项目 [HKUDS/nanobot](https://github.com/HKUDS/nanobot)。

当前仓库不是对该项目的直接搬运，而是基于 `Solon + Solon AI + 文件工作区` 重新实现的一套统一 Agent 运行时。后续理解和改造本项目时，应以当前仓库代码与测试为准。

## 文档目标

本文件面向当前 `SolonClaw` 仓库，帮助新的代理或开发者快速理解：

- 当前真实技术栈
- 运行时装配方式
- 会话/任务/子任务的行为边界
- 工作区、持久化、调试与渠道约束
- 修改代码时默认要遵守的协作规则

这不是 Solon 教程摘录，而是“基于当前代码状态”的项目协作说明。

## 当前技术栈

- Java `8`
- Solon `3.9.5`
- `solon-web`
- `solon-ai`
- `solon-ai-agent`
- `solon-ai-skill-cli`
- `solon-scheduling-simple`
- `solon-serialization-snack4`
- `solon-logging-logback-jakarta`
- `solon-test`
- Hutool `5.8.44`
- 钉钉 Stream SDK：`com.dingtalk.open:dingtalk-stream:1.1.0`
- 钉钉 OpenAPI SDK：`com.aliyun:dingtalk:1.5.59`

参考文档仍可看 `docs/Solon-v3.9.4.md`，但实际行为以当前代码和测试为准。

## 项目入口与装配

应用入口：

- [src/main/java/com/jimuqu/claw/SolonClawApp.java](D:/IdeaProjects/SolonClaw/src/main/java/com/jimuqu/claw/SolonClawApp.java)

统一装配入口：

- [src/main/java/com/jimuqu/claw/config/SolonClawConfig.java](D:/IdeaProjects/SolonClaw/src/main/java/com/jimuqu/claw/config/SolonClawConfig.java)
- [src/main/java/com/jimuqu/claw/config/SolonClawProperties.java](D:/IdeaProjects/SolonClaw/src/main/java/com/jimuqu/claw/config/SolonClawProperties.java)

当前装配特点：

- 项目自定义配置通过 `@BindProps(prefix = "solonclaw")` 绑定
- 运行时依赖统一在 `SolonClawConfig` 中以 `@Bean` 装配
- 长时资源统一走 `initMethod` / `destroyMethod`
- `@EnableScheduling` 已在应用入口启用

当前已接入生命周期管理的资源包括：

- `WorkspaceJobService`：启动时恢复持久化任务
- `DingTalkAccessTokenService`
- `DingTalkChannelAdapter`
- `HeartbeatService`

默认约定：

- 新增组件、控制器、配置类优先放在 `com.jimuqu.claw` 包下
- 第三方对象或复杂对象优先用 `@Configuration + @Bean`
- 普通业务对象优先保持容器托管，不要手动 `new`

## 当前核心架构

项目当前已经不是单纯的 Solon Web Demo，而是一套“统一运行时 + 多渠道适配 + 工作区驱动提示词 + 可派生子任务 + 可持久化定时任务”的 Agent 服务。

### 1. 统一消息模型

位于 [src/main/java/com/jimuqu/claw/agent/model](D:/IdeaProjects/SolonClaw/src/main/java/com/jimuqu/claw/agent/model) 及其子包：

- [src/main/java/com/jimuqu/claw/agent/model/envelope](D:/IdeaProjects/SolonClaw/src/main/java/com/jimuqu/claw/agent/model/envelope)
- [src/main/java/com/jimuqu/claw/agent/model/event](D:/IdeaProjects/SolonClaw/src/main/java/com/jimuqu/claw/agent/model/event)
- [src/main/java/com/jimuqu/claw/agent/model/route](D:/IdeaProjects/SolonClaw/src/main/java/com/jimuqu/claw/agent/model/route)
- [src/main/java/com/jimuqu/claw/agent/model/run](D:/IdeaProjects/SolonClaw/src/main/java/com/jimuqu/claw/agent/model/run)
- [src/main/java/com/jimuqu/claw/agent/model/enums](D:/IdeaProjects/SolonClaw/src/main/java/com/jimuqu/claw/agent/model/enums)

- `InboundEnvelope`：标准化后的入站消息
- `OutboundEnvelope`：标准化后的出站消息
- `ReplyTarget`：唯一可信的回复路由
- `AgentRun`：一次运行任务
- `ConversationEvent`：会话事件
- `RunEvent`：运行过程事件
- `LatestReplyRoute`：最近一次可回复外部路由
- `ChildRunSpawnedData` / `ChildRunCompletedData`：子任务事件载荷

硬规则：

- 回复路由只能来自 `ReplyTarget`
- 不允许根据“当前上下文”猜回复目标
- 渠道之间的 `sessionKey` 必须隔离，不能共享命名空间
- `SYSTEM` 类型消息不应覆盖最近一次真实外部会话路由

### 2. 运行时主链路

核心类位于 [src/main/java/com/jimuqu/claw/agent/runtime](D:/IdeaProjects/SolonClaw/src/main/java/com/jimuqu/claw/agent/runtime)：

- [AgentRuntimeService.java](D:/IdeaProjects/SolonClaw/src/main/java/com/jimuqu/claw/agent/runtime/AgentRuntimeService.java)
- [ConversationScheduler.java](D:/IdeaProjects/SolonClaw/src/main/java/com/jimuqu/claw/agent/runtime/ConversationScheduler.java)
- [SolonAiConversationAgent.java](D:/IdeaProjects/SolonClaw/src/main/java/com/jimuqu/claw/agent/runtime/SolonAiConversationAgent.java)
- [HeartbeatService.java](D:/IdeaProjects/SolonClaw/src/main/java/com/jimuqu/claw/agent/runtime/HeartbeatService.java)

当前实际流程：

1. 渠道或系统构造 `InboundEnvelope`
2. `AgentRuntimeService` 先做去重、写入会话事件、保存外部 `ReplyTarget`
3. 为该消息创建独立 `runId`
4. `ConversationScheduler` 按 `sessionKey` 控制会话级并发
5. `SolonAiConversationAgent` 基于历史、当前消息、工具和技能执行
6. 结果写入 `RunEvent` 和 `ConversationEvent`
7. 若允许对外回发，则通过原渠道回发

### 3. 会话并发与一致性规则

这是当前项目最重要的行为约束：

- 每条消息都是独立 run
- 并发控制是“按会话”，不是“全局串行”
- 单会话最大并发来自 `solonclaw.agent.scheduler.maxConcurrentPerConversation`
- 当前 `app.yml` 中有效值是 `4`
- `SolonClawProperties` 代码默认 `ackWhenBusy=true`，但当前 `app.yml` 覆盖为 `false`
- 当 `ackWhenBusy=true` 且该会话已有活跃任务时，系统会立刻发送“已收到”回执
- 历史重建按“用户消息顺序 + 已完成回复 + 可渲染系统事件”组织，不按完成时间倒灌重排
- 应用重启后，未完成 run 会被标记为 `ABORTED`

任何扩展都不能把系统退回成“全局单线程串行队列”。

### 4. 子任务与 continuation 机制

当前运行时支持把一个大任务拆成多个独立子任务。

相关能力：

- `spawn_task`
- `list_child_runs`
- `get_run_status`
- `get_child_summary`

实现特点：

- 子任务使用独立 `childSessionKey`
- 子任务应显式携带 `taskTitle` 与 `taskDescription`；标题用于日志、汇总和区分不同子任务
- 子任务 run 会记录 `parentRunId`、`parentSessionKey`、`parentReplyTarget`
- 子任务完成后，会向父会话写入结构化事件并自动触发一次 continuation run
- 父运行可进入 `WAITING_CHILDREN`
- 父运行派生子任务后，默认应先向用户说明已经安排了哪些子任务以及后续同步方式
- 子任务每次完成后，父会话都可以基于该次结果立即增量回复，不必默认等待全部子任务结束
- 只有在确实需要“全部结束后统一收口”的场景，才使用 `FINAL_REPLY_ONCE:` 前缀实现“仅发送一次最终聚合回复”
- `batchKey` 可用于给同一批子任务分组聚合

### 5. 主动通知能力

当前运行时支持在一次运行中主动向当前外部会话发送通知。

相关能力：

- `notify_user`
- `NotificationSupport`

边界：

- 只有当前会话已经绑定可用 `ReplyTarget` 时才能主动通知
- 主动通知会写入 `RunEvent`
- 主动通知不等于普通最终回复，两者可以分离

## 工作区与提示词系统

工作区由 [src/main/java/com/jimuqu/claw/agent/workspace](D:/IdeaProjects/SolonClaw/src/main/java/com/jimuqu/claw/agent/workspace) 负责：

- [AgentWorkspaceService.java](D:/IdeaProjects/SolonClaw/src/main/java/com/jimuqu/claw/agent/workspace/AgentWorkspaceService.java)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opensolon/solonclaw](https://github.com/opensolon/solonclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
