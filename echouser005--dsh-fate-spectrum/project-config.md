---
trigger: always_on
description: 本文件是 `dsh-fate-spectrum` 的长期协作总纲。它只定义元规则，不记录阶段进度，不复制具体契约，也不替代专项设计文档。
---

# AGENTS.md

本文件是 `dsh-fate-spectrum` 的长期协作总纲。它只定义元规则，不记录阶段进度，不复制具体契约，也不替代专项设计文档。

任何新 Agent 开始工作时，必须先读本文件，再按任务类型读取对应真相源。不得依赖聊天记忆、旧分支经验或第三方库的直觉行为。

> `dsh-fate-spectrum` 是 DeepSeek Harness（DSH）的确定性命理能力插件：插件计算事实，DSH 负责模型、Agent Loop、Session 与工具调度。凡是能由程序确定的历法和命盘事实，都不能交给模型猜测；凡是属于模型解释、会话状态或产品外壳的能力，都不能塞回计算核心。

## 一、产品方向

项目最终要形成一套可验证、可追溯、可组合的命理事实基础设施，让 DSH Agent 能围绕用户真实经历持续提供有依据的自我观察与行动参考。

能力沿同一技术主线逐步生长：

1. **命盘**：八字、紫微及其计算约定和来源信息。
2. **时间与作用**：干支历时间范围、原局与运限作用事实。
3. **流程与知识**：周报 Skill、经授权的案例与规则检索。
4. **交互呈现**：基于 DSH 可重放事件的命盘与局部分析组件。
5. **用户校准**：未来在明确授权和隐私审查后，通过 DSH Memory 记录已确认的现实反馈。

这些是完整愿景，不代表当前已实现范围。当前阶段、阻塞项和唯一下一步只以 [`docs/project/PROJECT_STATE.md`](docs/project/PROJECT_STATE.md) 为准。

## 二、真相源与阅读路由

同一规则只能有一个主要真相源。影响后续开发的决定必须写入仓库，不能只留在聊天中。

| 任务触发条件                                          | 开始前必须阅读                                                          |
| ----------------------------------------------------- | ----------------------------------------------------------------------- |
| 任何非简单任务                                        | `docs/project/PROJECT_STATE.md` 指向的活动里程碑和当前迭代、相关 ADR    |
| 判断长期产品方向或架构                                | `README.md`、`docs/architecture.md`、相关 ADR                           |
| 判断当前版本的功能范围、验收节点或完成定义            | `PROJECT_STATE.md` 指向的活动里程碑                                     |
| 判断当前版本的代码结构、文件职责、实现顺序或测试设计  | 活动里程碑链接的 `docs/develop/` 技术方案                               |
| 改分层、依赖方向、运行时边界、TypeScript 原则或技术栈 | `docs/architecture.md`、相关 ADR                                        |
| 改 Tool 参数、输出、状态、错误或模型可见内容          | `docs/tool-layer.md`、相关 ADR；修改前先获用户确认                      |
| 改共享出生、时间、八字或紫微原子能力输入输出          | 活动 develop 技术方案链接的原子能力审核稿、架构、Provider 探针证据      |
| 引入或升级 DSH/Cordis，改 Profile、Bundle、Tool 注册  | 锁定版本的 DSH 官方源码与文档，并运行真实组装 smoke                     |
| 引入或升级命理 Provider                               | 活动里程碑的探针要求、探针记录、许可证声明、差分与隔离测试              |
| 开始、结束或交接一轮开发                              | `DEVELOPMENT_WORKFLOW.md`、`PROJECT_STATE.md`、活动里程碑               |
| 准备 Tag、Release 或 npm 发布                         | `docs/project/DEVELOPMENT_WORKFLOW.md` 的发布闭环；必须获得当次明确授权 |

专项文档职责：

- [`README.md`](README.md)：只面向使用者表达插件使命、长期愿景、使用方式、版本能力、核心技术信念、隐私和边界；不得写仓库阶段、依赖版本、测试结果或开发进度。用户可见能力、安装方式、用法、版本、限制或隐私发生变化时，本轮必须评审 README；内部重构、依赖升级和测试变化不得自动改写它。
- [`docs/architecture.md`](docs/architecture.md)：长期架构、TypeScript 判断原则、依赖方向和演进护栏；不记录版本 TODO 或精确文件树。
- [`docs/tool-layer.md`](docs/tool-layer.md)：Tool、canonical outcome、错误与 Observation 契约。
- [`docs/project/PROJECT_STATE.md`](docs/project/PROJECT_STATE.md)：唯一的当前进度、阻塞项和下一步。
- [`docs/project/milestones/`](docs/project/milestones/)：跨多轮版本的用户功能目标、范围、验收节点、决策门和完成定义；不展开代码树和实现方案，同一时间只有一个活动里程碑。
- [`docs/develop/`](docs/develop/)：当前版本的开发技术方案，承载业务流水线、精确代码树、文件职责、实现顺序和分层测试；用户批准后才成为实现基线，版本完成后随里程碑冻结归档。
- [`docs/project/decisions/`](docs/project/decisions/)：用户确认过的长期决策；未决定事项不得写成“已接受”。
- [`docs/project/iterations/`](docs/project/iterations/)：每轮目标、改动、证据和遗留项。
- [`docs/project/CHANGELOG.md`](docs/project/CHANGELOG.md)：每轮开发完成后追加一句仓库变更摘要；它是开发索引，不是 README 产品版本史。
- [`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md)：依赖、固定版本、许可证和用途。

### README 更新边界

README 是使用者入口，不随每轮开发机械更新。修改前先判断触发了哪一栏；没有命中下表时，不得改 README。

| README 部分        | 何时更新                                                      | 依据与限制                                                                |
| ------------------ | ------------------------------------------------------------- | ------------------------------------------------------------------------- |
| 定义与愿景         | 产品使命、目标命主或长期价值发生变化，且用户明确确认          | 以用户确认的产品原文为主，只整理结构、逻辑和重复，不重新创作              |
| 长期规划／未来展望 | 长期能力方向被新增、删除或重新排序，且用户明确确认            | 只写方向；不得写成已经上线                                                |
| 当前能力           | 能力已经进入当前公开产品边界，并有端到端证据                  | 只描述命主能直接使用的能力，不写模块、测试或完成比例                      |
| 使用指南           | 真实 DSH 体验形成了可复用的安装步骤、对话案例或操作方式       | 未经实际体验和用户确认不得虚构；没有内容时保留 `[待补充]`                 |
| 最新版本           | 发布候选的用户可见能力已经验收，准备进入发布审核              | 用使用者语言说明这个版本能做什么；不得罗列代码、文件、异常分支或工程 Gate |
| 变更记录           | 一个公开产品版本确定或发布                                    | 每个版本只写一句产品变化；不得链接或复制 `docs/project/CHANGELOG.md`      |
| 核心技术           | 稳定的计算机制、Tool 分层或 Agent 交互机制发生实质变化        | 只写具体机制及作用；禁止口号、自我评价、开发过程和依赖堆砌                |
| 隐私与边界         | 数据采集、传输、日志、Session、Memory、风险声明或能力限制变化 | 必须与实际运行行为一致，宁可披露限制，不作超范围承诺                      |
| 开源许可           | License 或第三方许可义务变化                                  | 同步 `LICENSE` 与 `THIRD_PARTY_NOTICES.md`，许可证风险立即暂停            |

以下内容永远不进入 README：Phase、完成比例、测试结果、分支、提交、文件树、Provider 探针、依赖升级、内部 TODO、下一开发步骤和本轮工作汇报。

开发记录按职责分流：当前状态写 `PROJECT_STATE.md`，一轮的目标、改动和证据写 `iterations/`，每轮完成后的一句话摘要写 `docs/project/CHANGELOG.md`，长期决定写 ADR。任何开发记录都不能冒充 README 的产品版本更新。

DSH 处于开发者预览期。涉及 DSH 边界时，不能只读本仓库旧记录，必须核验当时最新的官方仓库、包清单、锁文件和对应专题文档：

- 官方仓库：<https://github.com/deepseek-ai/deepseek-harness>
- 开发与版本：<https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/development.md>
- DeepSeek Harness 架构：<https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/architecture.zh.md>
- Capability seams：<https://deepseek-harness.github.io/deepseek-harness/reference/capability-seams>
- Cordis 论文：<https://github.com/cordiverse/paper>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EchoUser005/dsh-fate-spectrum](https://github.com/EchoUser005/dsh-fate-spectrum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
