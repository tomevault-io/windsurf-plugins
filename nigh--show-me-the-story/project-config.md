---
trigger: always_on
description: > **重要**：当对项目进行任何修改（代码、配置、前端、提示词等）后，必须同步更新本文件，确保文档与项目实际情况完全一致。
---

# AGENTS.md — AI 小说生成器项目指南

> **重要**：当对项目进行任何修改（代码、配置、前端、提示词等）后，必须同步更新本文件，确保文档与项目实际情况完全一致。

## 项目概述

单二进制 Go Web 应用，Go 后端零外部依赖（仅标准库），通过 OpenAI 兼容 API 自动生成长篇小说。前端使用 Vite + Svelte 4 + DaisyUI 4 构建，产物通过 `embed.FS` 内嵌到二进制中。

- **Go 版本**：1.25.1
- **模块名**：`showmethestory`
- **默认端口**：`:48090`（可通过 `PORT` 环境变量覆盖）
- **前端**：Vite 5 + Svelte 4 + Tailwind CSS 4 + DaisyUI 5（xianii 暗色主题）
- **项目目录**：`storys/`（程序同目录下，每个故事项目一个子目录）
- **许可证**：MIT（见根目录 `LICENSE`）

## 编译与运行

```bash
# 完整编译（含前端构建）
task build                          # 推荐：自动 npm run build + go build

# 或手动分步
cd frontend && npm install && npm run build && cd ..   # 构建前端
go build -o show-me-the-story.exe .                    # 编译 Go（嵌入 frontend/dist/）

# 运行
./show-me-the-story.exe               # 运行（默认当前目录为项目目录）
./show-me-the-story.exe ./my-novel/   # 指定项目目录运行

# 开发模式
task dev:frontend                     # 启动 Vite dev server（热重载，端口 5173，代理 /api → :48090）
task dev                              # 编译并启动 Go 后端
```

编译前务必确认 `go build` 无报错。项目无测试框架，编译通过即为基本验证。

## 架构概览

```
用户浏览器 ←→ HTTP Server (web.go)
                  │
                  ├─ handlers.go    ← 所有 API 端点处理（含设定 CRUD、技能、聊天、去AI味）
│   ├─ 同步端点：直接返回 JSON
│   └─ 异步端点：tryStartTask() → go func() { defer h.endTask(); ... } → SSE 推送
                  │
                  ├─ SSE (logger.go) ← 实时日志/进度/事件推送到前端
                  │
                  ├─ outline.go     ← 大纲阶段逻辑 + EditChapterOutline
                  ├─ writing.go     ← 写作阶段逻辑 + 上下文注入 + 去AI味
                  ├─ foreshadow.go  ← 伏笔系统
                  ├─ continue.go    ← 续写功能（导入分析）
                  ├─ reconcile.go   ← 设定协调逻辑（AI 自动兼容新旧设定）
                  ├─ settings.go    ← 结构化设定（角色/世界观/组织/关系）CRUD + 持久化
                  ├─ skills.go      ← Skill 系统（内置 + 项目级，可选启用）
                  ├─ agent.go       ← Agent Loop 引擎 + 内置工具集（全局助理用）
                  ├─ chat.go        ← 会话管理（JSON 文件存储）
                  ├─ api.go         ← OpenAI API 调用 + 重试 + 致命错误检测 + context 支持
                  ├─ config.go      ← 配置结构体 + 加载/保存（含 SkillConfig）
                  ├─ state.go       ← 进度/章节/伏笔结构体 + 持久化
                  ├─ prompts.go     ← 提示词模板渲染 + 内置默认模板
                  ├─ postprocess.go ← 全书优化（诊断/核查/路线图/按章执行）+ postprocess.json 持久化
                  └─ filesys.go     ← 文件操作抽象
```

## 文件清单与职责

| 文件 | 职责 |
|------|------|
| `main.go` | 入口，确定程序目录（`progDir`），创建 `storys/` 目录，加载 API 配置，启动 Web 服务器（无项目选择状态） |
| `config.go` | `APIConfig`（含 `ContextBudgetTokens` 全书优化上下文预算）、`Config`（含 `SkillConfig`）、`StoryConfig`、`PromptsConfig` 结构体，Load/Save 函数，`applyDefaults` |
| `state.go` | `Progress`、`ChapterState`、`Foreshadow` 结构体，`LoadProgress`、`SaveProgress`（原子写入）、`ChapterMarkdownPath`、`SaveChapterMarkdown(projectDir, ...)`、`ForeshadowRoadmapPath`（项目目录 `Foreshadows.md`） |
| `api.go` | `CallAPI`/`CallAPIMessages`（同步）、`CallAPIStream`/`CallAPIStreamMessages`（流式，支持完整多轮消息历史）、`CallAPIWithRetry`/`CallAPIWithRetryLog`（无限重试）、`CallAPIStreamWithRetry`/`CallAPIStreamWithRetryLog`，`validateAPIConfig`、`isFatalAPIError`（401/403/404 致命，网络超时可重试） |
| `outline.go` | `generateOutline`、`reviseOutline`、`GenerateOutlineAction`（存在已确认章节时拒绝整体重新生成）、`ReviseOutlineAction`、`ConfirmOutlineAction`、`EditChapterOutline`、`cleanJSONResponse` |
| `writing.go` | `GenerateChapterAction`（含写前大纲一致性检查，共 5 步；第 5 步更新伏笔并落盘 `Foreshadows.md`）、`ReviseChapterAction`/`ReviseSpecificChapterAction`（修订后同步更新伏笔）、`ConfirmChapterAction`、`PolishChapterAction`、`SmoothTransitionsAction`（批量优化已确认章节衔接，逐章最小化重写开头、逐章落盘）、`parseFactCheckResult`（JSON 优先 + 字符串 fallback）、`checkOutlineConsistency`（写前检查本章大纲与已写剧情冲突，冲突时最小化修订本章大纲）、章节内容生成/摘要/事实核查/流式输出、`buildHistorySummary`、`buildPreviousChapterTail`（上一章尾部约 800 字注入写作 prompt）、`buildOutlineConstraints`（全书章节脉络反向约束：后续 10 章大纲防提前出现 + 前文大纲防一次性事件重复，注入写作与事实核查 prompt）、`appendIfMissingPlaceholder`（老项目持久化旧模板缺新占位符时把上下文块追加到渲染结果末尾兜底）、`splitChapterOpening` |
| `foreshadow.go` | `SuggestForeshadows`、`UpdateForeshadows`、伏笔格式化注入、伏笔告警、`BuildForeshadowRoadmapMarkdown`、`SaveForeshadowRoadmap`、`syncForeshadowsAfterChapter`、`NextForeshadowID` |
| `continue.go` | `AnalyzeExistingContent`、`ImportContinueAction`、`GenerateContinuationOutline`、`splitContentByChapters` |
| `reconcile.go` | `ReconcileSettingsAction`、`regeneratePendingOutlines`、设定协调逻辑 |
| `settings.go` | `Character`、`WorldviewEntry`、`Organization`、`Relation`、`ProjectSettings` 结构体，`LoadProjectSettings`、`SaveProjectSettings`、`buildCharacterContext`、`buildWorldviewContext` |
| `skills.go` | `Skill`、`SkillConfig` 结构体，`LoadBuiltinSkills`、`LoadProjectSkills`、`MergeSkills`、`GetEnabledSkills`、`GetEnabledSkillsByCategory`、`FormatSkillsContent`，`//go:embed embeds/skills` |
| `agent.go` | `Tool`、`AgentContext`、`AgentStep`、`ToolCall` 结构体，`RunAgentLoop`（多轮消息历史）、工具调用解析、内置工具集（读/写角色/世界观/章节等）、系统提示词含安全规则与工具选择指南、`requireConfirm`（破坏性工具需 `confirm: true`） |
| `chat.go` | `ChatSession`、`ChatMessage`、`ChatSessionIndex` 结构体，`LoadChatSessions`、`LoadChatSession`、`SaveChatSession`、`DeleteChatSession` |
| `postprocess.go` | `PostProcessState`/`RoadmapItem` 结构体，`LoadPostProcess`/`SavePostProcess`（`postprocess.json`）、`buildPostProcessBundle`（设定+摘要+全文组装与长文策略：全文/摘要模式）、`DiagnoseBookAction`、`ConsistencyCheckBookAction`（超长书按卷分段）、`BuildRoadmapAction`、`FullPostProcessAnalyzeAction`（诊断→核查→路线图）、`ExecuteRoadmapAction`（可选前置衔接优化 + 逐条定向修订/润色 + diff 节选） |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nigh/show-me-the-story](https://github.com/Nigh/show-me-the-story) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
