---
trigger: always_on
description: > **最后更新**：2026-07-08
---

# langchain4android AI Agent 系统：唯一事实来源 (SSOT)

> **版本**：2.0（合并版）  
> **状态**：进行中  
> **最后更新**：2026-07-08  
> **维护者**：CO Agent  
> **历史合并说明**：本文档由根目录 `AGENTS.md` 与 `agents/README.md` 合并而成。`agents/README.md` 中的角色速查、状态板模板、Token 节省、回流机制、Tools 输入输出、工具调用速查等内容已并入本文档对应章节或附录，原文件已删除。

> 本文档为**顶层治理文档**，定义 Agent First 的研发流程与协作规范。
>
> **langchain4android** 是面向 Android 的 AI Agent 基础库（Java），Demo 工程 **PoLang（破浪相册）** 验证其在真实场景中的可行性。

---

## 1. 项目背景：Agent First 三重实验

langchain4android 是一个元实验（meta-experiment），同时探索三个层次：

| 层次 | 实验对象 | 核心问题 |
|------|----------|----------|
| **基础库** | langchain4android（:agent-core） | LangChain4j 风格 API 能否在 Android 高效运行？ |
| **运行时** | PoLang Agent 编排层（:app） | LLM 能否成为应用的中枢神经系统？ |
| **架构层** | Agent First 客户端框架 | 什么样的架构让 Agent 最高效？ |
| **流程层** | Agent First 研发流程 | Agent 如何通过编排 Tools 完成开发？ |

**核心假设**：当基础设施原子化为 Tools 层后，Agent 可以从「辅助工具」进化为「主导力量」。

---

## 2. Agent First 的代码架构原则

langchain4android 的所有代码遵循以下原则，确保 Agent 能高效理解、修改、验证：

### 2.1 显式优于隐式（Explicit > Implicit）

```kotlin
// ❌ 隐式依赖：AI 需要全局搜索理解生命周期
object BeautyEngine {
    fun getInstance() = instance
}

// ✅ 显式注入：构造函数即文档
class CameraViewModel(
    private val beautyEngine: BeautyEngine,
    private val agentUseCase: AiAgentUseCase,
    private val settingsRepository: SettingsRepository
) : ViewModel()
```

**收益**：通过构造函数签名，AI 即可理解组件协作关系，无需跨文件搜索。

### 2.2 枚举优于条件（Exhaustive > Conditional）

```kotlin
// ❌ 布尔标志组合爆炸
class CameraState(
    val isLoading: Boolean,
    val hasError: Boolean,
    val isPreviewing: Boolean
)

// ✅ 枚举所有合法状态
sealed interface CameraState {
    data object Initializing : CameraState
    data class Previewing(val settings: BeautySettings) : CameraState
    data class Error(val reason: String) : CameraState
}
```

**收益**：状态空间显式编码，AI 可枚举所有边界情况，不会遗漏。

### 2.3 自描述优于注释（Self-Describing > Commented）

```kotlin
// ❌ 注释与代码可能脱节
// 调节美颜参数
fun adjust(params: Map<String, Int>) // AI 不知道有哪些参数

// ✅ 类型系统即文档
data class BeautyParameters(
    val smooth: IntRange = 0..100,
    val whiten: IntRange = 0..100,
    val slimFace: IntRange = -50..50
)
fun adjust(params: BeautyParameters) // 类型即契约
```

**收益**：类型系统强制一致性，AI 可靠类型推导而非易腐烂的注释。

### 2.4 结构化可观测性（Structured Observability）

```kotlin
// ❌ 纯文本日志，需正则解析
Log.d("Camera", "Agent parsed: $input -> $intent")

// ✅ 结构化事件，AI 可直接消费
data class AgentCommandParsedEvent(
    val rawInput: String,
    val parsedIntent: Intent,
    val confidence: Float,
    val timestamp: Long
) : LogEvent

Logger.log(AgentCommandParsedEvent(...))
```

**收益**：结构化日志可被 AI 消费，实现自我诊断和自我改进。

> **实现状态（2026-06）**：结构化可观测性为架构设计愿景。实际代码中目前以 `PicMe:` 前缀标签 + `Log.d/w/e` 为主要日志形式，结构化事件（如 `AgentCommandParsedEvent`）尚未在全局范围强制要求。这是后续 Phase 3 的重点推进方向。

---

## 3. Agent 角色与协作流程

PoLang 采用**角色化协作模型**：每个 Agent 角色有明确的职责边界、输入输出契约。

### 3.1 角色定义

| 角色 | 标识 | 核心职责 | 关键能力 | 参考文档 | 激活方式 |
|------|------|----------|----------|----------|----------|
| **[CO]** 协调者 | `🤖CO` | 任务分级、状态板维护、流程推进 | 复杂度分析、状态板维护 | [`agents/co_agent.md`](agents/co_agent.md) | **所有请求默认激活** |
| **[PM]** 产品经理 | `🤖PM` | 需求澄清、PRD 维护、验收标准 | 需求拆解、文档同步 | [`agents/pm_agent.md`](agents/pm_agent.md) | 由 CO 在需求类任务中激活 |
| **[RD]** 全栈工程师 | `🤖RD` | 端到端实现、Self-Heal、Tools 编排 | 代码生成、Tools 编排 | [`agents/rd_agent.md`](agents/rd_agent.md) | 由 CO 在实现类任务中激活 |
| **[CR]** 规范守护者 | `🤖CR` | 架构合规审查、代码质量裁决 | 红线检查、影响分析 | [`agents/review_agent.md`](agents/review_agent.md) | 由 CO 在 RD 完成后激活 |
| **[QA]** 质量专家 | `🤖QA` | 边界测试、性能基线、端到端验收 | 场景设计、回归检测 | [`agents/qa_agent.md`](agents/qa_agent.md) | 由 CO 在 CR 通过后激活 |

**设计原则**：
- 每个角色有**明确的输入输出契约**
- 每个角色有**可验证的交付标准**
- 角色间通过 **CO 协调**传递信息，非直接沟通
- **CO 是所有用户请求的唯一入口**

### 3.2 协作流程（CO 驱动）

```
用户请求
    ↓
[CO] 分析任务类型 → 复杂度分级（L1/L2/L3）→ 创建状态板
    ↓
[PM] 需求对齐 → 输出可执行结论（AC）
    ↓
[RD] 原子化实现 → 代码 + 文档同步
    ↓  调用 Tools 完成验证
[RD] Self-Heal 闭环 → 编译 → 安装 → 测试 → 日志
    ↓  [CO 检测到"编译通过"自动推进]
[CR] 规范审查 → 架构合规、代码质量
    ↓  [CO 检测到"审计通过"自动推进]
[QA] 验收测试 → 边界、性能、体验
    ↓  [CO 检测到"验收通过"自动推进]
[CO] 汇总交付 → 更新状态板 → 报告闭环
```

**CO 推进规则**：
- RD 报告编译通过 → CO **必须**立即启动 CR 审计
- CR 报告无 Critical → CO **必须**立即启动 QA 验收
- QA 报告无 P0 缺陷 → CO **必须**立即生成最终交付报告
- **严禁**在 L1/L2 任务中间环节要求用户确认

### 3.3 Tools 层

基础设施原子化为 **Tools**，供 Agent 编排调用：

| Tool | 功能 | 输入 | 输出 | 调用者 | 状态 |
|------|------|------|------|--------|------|
| `CompileTool` | 代码编译检查 | 源码变更 | 编译结果/错误日志 | RD | 🔄 脚本实现 (`./gradlew`) |
| `InstallTool` | 安装到设备 | APK | 安装状态 | RD | 🔄 脚本实现 (`adb install`) |
| `ScreenshotTool` | 自动截屏 | 设备连接 | 截图文件 | RD/QA | 🔄 脚本实现 (`adb screencap`) |
| `LogAnalysisTool` | 结构化日志分析 | Logcat | 结构化事件 | RD | 📋 设计愿景 |
| `DocSyncTool` | 文档同步检查 | Git diff | 需更新文档列表 | CR | 📋 设计愿景 |
| `ScreenshotDiffTool` | UI 回归检测 | 截图对比 | Diff 报告 | QA | 🔄 脚本实现 (`screenshot-diff.py`) |
| `PerfBaselineTool` | 性能基线对比 | 性能指标 | 对比报告 | QA | 📋 设计愿景 |

> **实现状态（2026-06）**：Tools 层概念已定义，但大部分以独立 shell 脚本（`./scripts/`）或 Gradle task 形式存在，尚未封装为统一的 Agent-tools 接口。`ScreenshotDiffTool` 等已有对应脚本落地。

**关键转变**：从「人类操作脚本」到「Agent 编排 Tools」。

### 3.4 触发口令与执行模式

| 口令 | 模式 | 自动化程度 | CO 行为 | 适用场景 |
|------|------|-----------|--------|----------|
| （无口令） | **默认模式** | L1 全自动 / L2 半自动 | 自动分析分级并启动对应流程 | 日常开发任务 |
| `自动执行` | 全链路自动 | L1/L2 全自动 | 强制启动完整 CO→PM→RD→CR→QA 流程 | 明确的全链路需求 |
| `保守执行` | 全链路可控 | 关键节点暂停 | 每阶段完成后暂停等待用户确认 | 高风险变更、不可逆操作 |
| `仅分析` | 诊断模式 | 不执行 | CO 仅输出分析，不启动任何角色 | 需求澄清、方案比选 |

**默认模式分级行为**：
- **L1 任务**（单文件修改、已知模式）：CO→RD→CR→QA，全自动推进，仅最终报告
- **L2 任务**（跨多文件、新功能）：CO→PM→RD→CR→QA，半自动，关键节点简报

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [littleseven/langchain4android](https://github.com/littleseven/langchain4android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
