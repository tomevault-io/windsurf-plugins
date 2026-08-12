---
trigger: always_on
description: > **最后更新**：2026-08-09
---

# polang AI Agent 系统：唯一事实来源 (SSOT)

> **版本**：2.4（仓库重组版）  
> **状态**：生效中  
> **最后更新**：2026-08-09  
> **维护者**：项目开发者  
> **历史合并说明**：本文档由根目录 `AGENTS.md` 与 `agents/README.md` 合并而成。
>
> 2026-08-03 更新：移除已退役的 CO/PM/RD/CR/QA 角色协作管线（`agents/*_agent.md` 已删；该管线从未被 kimi 实际调度，kimi 改用全局子代理）。本文档聚焦**架构原则、全局红线、文档治理与工具脚本**，不再定义强制角色编排流程。
>
> 2026-08-07 更新：仓库重组 Phase 3——项目改名 polang（原 langchain4android），`app/` → `androidApp/`（`:app` → `:androidApp`），引擎模块迁入 `engines/`（`:beauty-api`/`:beauty-engine`/`:mnn-core`/`:sentencepiece` → `:engines:*`），Gradle `rootProject.name` = "polang"。

> 本文档为**顶层治理文档**，定义 Agent First 的研发规范。
>
> **polang** 是 PoLang（破浪相册）应用的 Monorepo：Android 应用为主体，KMP 跨端改造进行中。

---

## 1. 项目背景：Agent First 三重实验

polang 是一个元实验（meta-experiment），同时探索三个层次：

| 层次 | 实验对象 | 核心问题 |
|------|----------|----------|
| **基础库** | LangChain4j 风格 Android Agent 基础库（原 `:agent-core` fork，已删除） | LangChain4j 风格 API 能否在 Android 高效运行？（结论：自维护 vendored fork 不可持续——冻结上游、死重多、0 测试；2026-08 全面迁移至 JetBrains Koog，fork 模块已删除） |
| **运行时** | PoLang Agent 编排层（`:shared` KMP 模块 + `:androidApp` 组合根） | LLM 能否成为应用的中枢神经系统？ |
| **服务端** | PoLang Server（`server/` Ktor 后端） | AI 网关、账号体系、管理后台能否支撑端侧 Agent？ |
| **架构层** | Agent First 客户端框架 | 什么样的架构让 Agent 最高效？ |
| **流程层** | Agent First 研发流程 | Agent 如何通过编排 Tools 完成开发？ |

**核心假设**：当基础设施原子化为 Tools 层后，Agent 可以从「辅助工具」进化为「主导力量」。

---

## 2. Agent First 的代码架构原则

polang 的所有代码遵循以下原则，确保 Agent 能高效理解、修改、验证：

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

> **实现状态（2026-07-26）**：结构化可观测性已有首个落地件——Agent 终端运行感知层三件套（`polang_llm_log.db` 的 `llm_call_log` 推理层 / `tool_call_log` 行动层 / `js_run_log` 端侧 JS 沙盒执行层），事件模型引擎无关、可被 AI 消费（详见 `docs/superpowers/specs/2026-07-26-js-sandbox-observability-design.md`）。其余模块仍以 `PoLang:` 前缀标签 + `Log.d/w/e` 为主，结构化事件（如 `AgentCommandParsedEvent`）尚未在全局范围强制要求，是后续 Phase 3 的推进方向。

---

## 3. 工具与自动化

基础设施原子化为 **Tools**，供 AI 编排调用，并通过脚本形成闭环验证。

### 3.1 Tools 层

| Tool | 功能 | 输入 | 输出 | 状态 |
|------|------|------|------|------|
| `CompileTool` | 代码编译检查 | 源码变更 | 编译结果/错误日志 | 🔄 脚本实现 (`./gradlew`) |
| `InstallTool` | 安装到设备 | APK | 安装状态 | 🔄 脚本实现 (`adb install`) |
| `ScreenshotTool` | 自动截屏 | 设备连接 | 截图文件 | 🔄 脚本实现 (`adb screencap`) |
| `LogAnalysisTool` | 结构化日志分析 | Logcat | 结构化事件 | 📋 设计愿景 |
| `DocSyncTool` | 文档同步检查 | Git diff | 需更新文档列表 | 📋 设计愿景 |
| `ScreenshotDiffTool` | UI 回归检测 | 截图对比 | Diff 报告 | 🔄 脚本实现 (`screenshot-diff.py`) |
| `PerfBaselineTool` | 性能基线对比 | 性能指标 | 对比报告 | 📋 设计愿景 |

> **实现状态（2026-07）**：`CompileTool`、`InstallTool`、`ScreenshotTool` 已通过 Gradle 脚本和 adb 命令落地；`ScreenshotDiffTool` 已有 `scripts/screenshot-diff.py` 实现；`LogAnalysisTool`、`DocSyncTool`、`PerfBaselineTool` 仍为设计愿景，待 Phase 3 基础设施完善。

**关键转变**：从「人类操作脚本」到「AI 编排 Tools」。

### 3.2 自动化脚本

| 脚本 | 用途 |
|------|------|
| `./scripts/ai-gate.sh` | 代码质量门禁 |
| `./scripts/auto-dev-loop.sh` | 编译→安装→启动→截屏→日志 |
| `./scripts/impact-analyzer.sh` | 变更影响分析 |
| `./scripts/doc-sync-guardian.sh` | 文档同步检查 |
| `./scripts/test-generator.py` | 基于 public 方法生成测试骨架 |
| `./scripts/screenshot-diff.py` | UI 回归检测 |

> **闭环验证习惯**：代码改动后走「编译 → 安装 → 测试 → 日志」闭环（`auto-dev-loop.sh`）；失败时基于日志定位根因再修，单任务自动重试最多 2 次，不盲目堆尝试。

**收益**：标准化工具消除人工操作的不确定性，AI 可编排完成复杂验证。

### 3.3 Token 优化

- 推进消息简短，聚焦增量信息，不重复已知上下文。
- 用 `TodoWrite` 追踪任务进度，替代长篇文字汇报。

### 3.4 工作区隔离（强制）

- 任何代码改动任务开工前，**必须先建隔离工作区**：检测当前是否已在 worktree；不在则在 `.worktrees/` 下创建独立 worktree + 专用分支（遵循 `using-git-worktrees` skill），征得用户同意后动工
- **禁止**在承载未提交改动或不相干特性分支的当前工作区直接改代码
- 提交前确认分支归属：fix/feat 只落到自己的专用分支，绝不混入其他特性分支的历史
- 工作区已存在不属于本任务的未提交改动时，只 `git add` 本任务相关文件，其余保持不动

### 3.5 AI 工具模型分工（逻辑档位）

强/弱是相对的——本项目用「**逻辑档位 + 各工具自绑物理模型**」解耦：角色只声明逻辑档，物理模型各工具自配，换模型只改绑定、不动角色。

| 逻辑档 | 含义 | Claude Code 绑定 | kimi-code 绑定 |
|--------|------|------------------|----------------|
| **STRONG** | 复杂推理:架构/评审/调试/规划 | glm-5.2(Fable 档) | K3(primary) |
| **WEAK** | 便宜 fan-out:搜索/探索/梳理/摘要 | glm-5.1(Haiku 档) | glm-5.2(secondary) |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [littleseven/polang](https://github.com/littleseven/polang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
