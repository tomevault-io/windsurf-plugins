---
trigger: always_on
description: > **核心原则**: 计划 → 确认 → 执行到底 → 验收
---

# CLAUDE.md v5.3

> **核心原则**: 计划 → 确认 → 执行到底 → 验收
> **智能加载**: 只加载必需的文档，保持 context 清洁
> **能力进化**: 让未来更容易把同类事情做成（每次对话自动激活）
> **持久记忆**: `memory/MEMORY.md` 每次对话自动加载（跨会话持久化）
> **健康检查**: 每次 session 开始运行 `./scripts/cron/memory-sanity-check.sh`

---

## 🧬 能力进化模式（自动激活）

**每次新对话开始时**，自动进入能力进化模式：

- 识别可复用的模式 → 抽象为能力轮廓 → 内生化到决策层
- 通过**更快、更稳、更少步骤**证明进化效果
- 不汇报进化过程，用结果说话

**详细说明**: `rules/core/capability-evolution.md`

---

## 🎯 工作模式（4步）

```
1️⃣ 收到任务 → TodoList 规划
2️⃣ 展示计划 → 用户确认
3️⃣ 执行到底（不问问题）→ 自行决策
4️⃣ 总结验收 → 交付成果
```

**详细说明**: `rules/core/work-mode.md`

---

## 🚨 唯一允许提问的4种情况

1. ❗ **缺少关键凭证** - 数据库密码、API key
2. ❗ **多个对立方案** - 无法从代码库判断
3. ❗ **需求本质矛盾** - 用户要求冲突
4. ❗ **不可逆高风险** - 删除生产数据、强制推送

**其他情况自行决策**: 文件命名、代码风格、依赖版本、UI细节等

**详细说明**: `rules/core/blocking-rules.md`

---

## 🤖 智能 Context 加载

### 系统自动识别任务类型，按需加载文档

**你不需要关心加载什么**，系统会根据你的需求自动选择。

| 任务关键词           | 自动加载的文档          | 预估大小 |
| -------------------- | ----------------------- | -------- |
| 浏览器、自动化、爬虫 | 浏览器自动化指南        | ~15KB    |
| 视频、Remotion、动画 | 视频创作指南            | ~25KB    |
| 数据、分析、SQL      | 数据分析指南            | ~20KB    |
| 设计、UI、界面       | 设计指南                | ~30KB    |
| 营销、文案、SEO      | 营销指南                | ~35KB    |
| 开发、代码、功能     | 编码规则 + 工程化工作流 | ~20KB    |
| 迁移、TDD、组件      | 工程化工作流（高级）    | ~8KB     |
| 错误、bug、调试      | 错误目录                | ~12KB    |
| 安全、漏洞、审计     | 安全规则                | ~15KB    |

**系统说明**: `CONTEXT_MANAGER.md`

---

## 🚀 快速开始（我想要...）

### 不知道用什么工具？

👉 **查看快速决策树**: `index/task-router.md`

- 30秒找到你需要的工具
- 按任务类型分类
- 包含所有能力的快速链接

### 常用能力快速跳转

| 能力            | 快速链接                                           |
| --------------- | -------------------------------------------------- |
| 🌐 浏览器自动化 | `capabilities/browser-automation/decision-tree.md` |
| 🎬 视频创作     | 项目级 `.claude/rules/remotion-auto-production.md` |
| 📊 数据分析     | `capabilities/data-analysis-guide.md`              |
| 🎨 UI 设计      | `design/DESIGN_MASTER_PERSONA.md`                  |
| 📝 营销内容     | `vibe-marketing/VIBE_MARKETING_GUIDE.md`           |
| 🐛 错误调试     | `errors/top-5-errors.md`                           |
| 🤖 Agent 编排   | `rules/agents.md`                                  |

---

## ⚠️ Top 5 高频错误（快速参考）

| 错误     | 核心问题   | 快速检查                             |
| -------- | ---------- | ------------------------------------ |
| **E001** | 异步未并行 | 多个异步操作是否用 `Promise.all()`？ |
| **E002** | 轮询无超时 | 轮询是否设置 `maxAttempts`？         |
| **E003** | 错误未抛出 | `catch` 块是否 `throw error`？       |
| **E004** | SQL未用CTE | JOIN 后过滤 → 用 CTE 预过滤大表      |
| **E007** | 资源泄漏   | 所有退出路径都清理资源？             |

**完整错误目录**: `errors/ERROR_CATALOG.md` (E001-E015)

---

## 🧠 核心方法论（长任务）

### 三文件模式

```
task_plan.md     - 任务规划和进度追踪
notes.md         - 研究笔记和发现记录
[deliverable].md - 最终产出物
```

**关键**: 每个重要决策点前重新读取 `task_plan.md`

### 阶段门控

```
Phase 1: 需求理解 → [用户确认 "ready"]
Phase 2: 设计方案 → [确认]
Phase 3: 实现代码
```

**原则**: 永远不进入下一阶段，直到用户明确确认

---

## 🔧 能力库（按需加载）

### MCP Servers（外部数据）

| 任务     | MCP        | 文档                               |
| -------- | ---------- | ---------------------------------- |
| SQL查询  | bytebase   | `capabilities/mcp-servers.md`      |
| 浏览器   | playwright | `capabilities/browser-automation/` |
| 监控日志 | honeycomb  | `capabilities/mcp-servers.md`      |

### Skills（自动化任务）

| 类别      | 示例                | 文档                                     |
| --------- | ------------------- | ---------------------------------------- |
| Git工作流 | /commit, /create-pr | `capabilities/skills-guide.md`           |
| 测试生成  | /write-tests        | 同上                                     |
| UI设计    | ui-ux-pro-max       | 同上                                     |
| 营销      | 24个专业Skills      | `capabilities/MARKETING_SKILLS_GUIDE.md` |

**完整清单**: `capabilities/skills-guide.md` (81个Skills)

---

## 📚 完整文档导航

### 索引层（快速查找）

- `index/task-router.md` - 任务路由决策树（30秒找到工具）
- `index/capabilities-index.md` - 能力索引
- `index/tools-index.md` - 工具索引
- `index/error-patterns-index.md` - 错误模式索引

### 记忆系统（v5.4 增强）

- `memory/MEMORY.md` - **持久化记忆Hub**（每次对话自动加载，<200行）
- `memory/engineering-patterns.md` - 工程模式详细记录
- `memory/project-contexts.md` - 项目状态追踪
- `memory/tools-and-services.md` - 工具与服务配置
- `memory/learned-capabilities.md` - **已内生化的能力**（能力进化持久化）
- `memory/archive/` - 过期内容归档（7天以上自动归档）
- `HEARTBEAT.md` - **任务追踪**（活跃任务、阻塞项、系统健康）

### 同步与健康检查

- `scripts/cron/memory-sanity-check.sh` - 检测冲突/膨胀/未推送
- `scripts/cron/daily-git-sync.sh` - 带告警的同步（不吞错误）

### 规则库（自动加载）

- `rules/core/` - 核心规则（总是加载）
  - `capability-evolution.md` - **能力进化模式（每次对话自动激活）**
  - `blocking-rules.md`, `work-mode.md`
- `rules/domain/` - 领域规则（按需加载）
  - `coding.md`（含 Common Patterns）, `testing.md`, `security.md`, `git.md`, `engineering-workflows.md`
- `rules/evomap/` - EvoMap 规则（合并收束后）
  - `evomap-guide.md` - 统一指南（模式+工作流+经济模型）
  - `evomap-content-guidelines.md` - 内容优化（8000字符限制）
- `rules/agents.md` - Agent 编排
- `rules/hooks.md` - Hooks 系统
- `rules/performance.md` - 性能优化

### 能力库（按需加载）

- `capabilities/browser-automation/` - 浏览器自动化
- `capabilities/video-creation/` - 视频创作
- `capabilities/data-analysis/` - 数据分析
- `design/` - UI 设计
- `vibe-marketing/` - 营销内容

### 错误库（按需加载）

- `errors/top-5-errors.md` - 高频错误快速参考
- `errors/ERROR_CATALOG.md` - 完整错误目录 (E001-E015)

### 知识库（参考）

- `KNOWLEDGE_MAP.md` - 知识图谱（12个Mermaid图）
- `QUICK_REFERENCE.md` - 一页速查表

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Arxchibobo/claude-Reconstruction](https://github.com/Arxchibobo/claude-Reconstruction) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
