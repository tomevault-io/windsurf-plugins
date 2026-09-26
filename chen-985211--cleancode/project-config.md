---
trigger: always_on
description: 本文是 cleancode 仓库的 AI 阅读入口，只负责把当前任务路由到必要文档。
---

# AGENTS.md

## 定位

本文是 cleancode 仓库的 AI 阅读入口，只负责把当前任务路由到必要文档。

完整文档目录由 [文档中心](docs/README.md) 维护。本文不复制开发、架构、测试或产品规则，也不要求 AI 在每个回合重复读取固定文档集合。

## 阅读原则

1. 先理解用户目标、列出已知目标路径并检查直接目标文件，再判断本次实际动作。
2. 必读文档由“目标路径基础路由”和“动作与风险叠加路由”共同决定；多个条件同时命中时读取并集，不得只选择其中一种归类。
3. 只读取会影响本次判断、修改或验证的文档和章节，不做预防性全库阅读。
4. 文档中的普通链接只表示导航、出处或事实移交，不自动产生继续阅读义务。只有当前任务也命中链接目标的路由条件，或当前文档不足以解决事实冲突时，才继续读取。
5. 同一连续任务中，已经读取且内容未变化的文档不重复读取；复用时必须在开工回执中如实注明。任务目标、目标路径或实际动作变化后重新路由。
6. 修改规则文档时，读取目标文档及本次实际改变的规则 owner；不得因为目标文档包含链接就递归读取所有被引用文档。

默认读取相关章节即可。只有任务会改变整份文档的职责、存在跨章节冲突，或无法通过局部内容确定规则时，才完整阅读该文档。

## 开工要求

只读分析、解释、审查和状态查询不要求读取开发协作规范，也不要求开工回执；当结论依赖项目规则时，仍须按本文路由读取必要的 owner 文档。

任务会修改项目文件、配置、依赖、Git 状态或其他项目状态时，必须先读取 [开发协作规范](docs/engineering/development.md) 中与任务分级、执行流程、验证和输出相关的章节，并按其要求输出包含规则路由证据的开工回执。

需要修改文件但尚未完成当前任务所需阅读时，不得开始修改、运行测试或创建提交。

用户要求“直接开始”、跳过 SDD、跳过 Spec/Plan 或不等待确认时，只影响开发协作规范允许省略的流程，不取消本文的文档路由、开工回执、适用的 TDD 或验证要求。

## 确定性路由流程

在执行会改变项目状态的动作前，必须按以下顺序完成路由：

1. 列出当前已知的目标文件、目录或配置。
2. 按“目标路径基础路由”确定基础必读文档。
3. 按“动作与风险叠加路由”加入本次行为实际命中的文档。
4. 在开工回执中记录“目标路径或动作 → 命中文档及章节”，然后才能修改、运行测试或提交。
5. 执行中新增目标路径、跨入新层级或发现新的行为风险时，先补充路由和开工回执，再处理新增范围；已经完成且未变化的路由无需重复。

### 目标路径基础路由

| 目标路径或文件类型                                                                                                                                                        | 基础必读文档                                                                                                                                                                                                                        |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 任意 `src/**` 生产代码                                                                                                                                                    | [架构文档](docs/engineering/architecture.md)的“核心原则”“分层规则”和目标所属“层级职责”；同时读取[测试规范](docs/testing/testing.md)以确定最低有效测试层。仅注释、纯格式化或不改变契约的机械生成物可以免读测试规范，并在回执说明理由 |
| `src/shared-kernel/**`                                                                                                                                                    | [架构文档](docs/engineering/architecture.md)的领域边界、跨上下文协作和顶层结构章节；[上下文地图](docs/engineering/context-map.md)                                                                                                   |
| `src/contexts/project/**`                                                                                                                                                 | [项目与分支工作区生命周期](docs/contexts/project/workspace-lifecycle.md)                                                                                                                                                            |
| `src/contexts/block-graph/**`                                                                                                                                             | [积木图模型](docs/contexts/block-graph/block-graph.md)；动作目标、审批作用对象或 Agent 工具同时命中时再读[积木动作模型](docs/contexts/block-graph/block-action-model.md)                                                            |
| `src/contexts/canvas-arrangement/**`                                                                                                                                      | [画布视觉整理](docs/contexts/canvas-arrangement/canvas-arrangement.md)；涉及终端/组合或 Agent 自身位置 owner 时继续读取对应 BlockGraph 或 Agent 专文                                                                                |
| `src/contexts/run/infrastructure/{pty,provider,persistence,terminal-model,filesystem}/**`，或 Run 中名称包含 `Terminal`、`Session`、`Recovery`、`WorkingDirectory` 的目标 | [终端会话生命周期](docs/contexts/run/terminal-session.md)                                                                                                                                                                           |
| `src/contexts/run/infrastructure/{block-graph,readiness}/**`，或 Run 中名称包含 `Workflow`、`Task`、`ManagedService` 的目标                                               | [终端依赖工作流](docs/contexts/run/terminal-workflow.md)                                                                                                                                                                            |
| `src/contexts/run/infrastructure/network/**`，或 Run 中名称包含 `Port`、`Lease`、`Endpoint` 的目标                                                                        | [本地服务端口治理](docs/contexts/run/service-port-management.md)                                                                                                                                                                    |
| 其他 `src/contexts/run/**`                                                                                                                                                | 根据实际动作至少选择一份 Run owner 专文，并在回执写明选择依据；命中多种 Run 语义时读取并集                                                                                                                                          |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chen-985211/cleancode](https://github.com/chen-985211/cleancode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
