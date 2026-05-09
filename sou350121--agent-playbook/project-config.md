---
trigger: always_on
description: 系统治理文件 — 适用于 Claude Code、Codex、Cursor 等 AI 编码智能体
---

# AGENTS.md

系统治理文件 — 适用于 Claude Code、Codex、Cursor 等 AI 编码智能体

> **最高准则：** 在开始任何工作前，务必阅读并严格遵守 [AGENT_CONSTITUTION.md](./AGENT_CONSTITUTION.md)。

---

## 1. 系统心智模型

**Agent-Playbook = AI App 生态的工程级记忆库**

本仓库不是笔记集，而是一个**有结构的知识操作系统**，服务于对 AI 应用生态的持续跟踪与工程化理解。

三层架构：

```
landscape（生态地图）
    ↓ 知道有什么工具、谁在主导方向
theory（知识骨架）
    ↓ 理解底层原理、设计方法、工程实践
playbooks（执行手册）
    ↓ 知道怎么做、有哪些模板和用例
```

theory/ 六模块结构：

```
01-principles/   底层原理     — 机器如何「想」
02-agent-design/ Agent 设计   — 记忆/角色/工具/多 Agent
03-engineering/  工程实战 ★  — Agentic 系统生产交付全链路（三支柱）
04-paradigm/     范式转变     — 识别不可逆的思维方式切换
05-strategy/     战略生存     — 工程师定位、变现、进化路径
06-frontier/     前沿研究     — 与工程最近的前沿突破
```

03-engineering 三支柱（并列，缺一不可）：

```
支柱 A 护欄与安全   — 物理/逻辑/流程/自动化 + 失效分类 + 信任层级
支柱 B Context 工程 — 上下文注入 + Spec 驱动 + 委派设计 + 控制平面
支柱 C 评估与迭代   — Eval 体系构建 + Eval 持续运维 + Ralph Loop
```

**memory/ 目录由 Pulsar 流水线自动写入，智能体绝对不得触碰。**

---

## 2. 文件写权限矩阵

| 路径 | 操作 | 规则说明 |
|------|------|---------|
| `landscape/app-index.md` | **仅追加** | 禁止删除条目、禁止重排顺序、禁止批量重写 |
| `landscape/technology-landscape.md` | 完整编辑 | 可自由增改，保持 section 结构一致 |
| `landscape/influencers.md` | 完整编辑 | 可自由增改，保持人物卡片格式 |
| `theory/01-principles/*.md` | 新建 / 编辑 | 遵守 §6 写作规范；底层原理类内容 |
| `theory/02-agent-design/*.md` | 新建 / 编辑 | 遵守 §6 写作规范；Agent 设计类内容 |
| `theory/03-engineering/*.md` | 新建 / 编辑 | 遵守 §6 写作规范；Agentic 工程实战类内容 |
| `theory/04-paradigm/*.md` | 新建 / 编辑 | 遵守 §6 写作规范；范式转变类内容 |
| `theory/05-strategy/*.md` | 新建 / 编辑 | 遵守 §6 写作规范；战略定位类内容 |
| `theory/06-frontier/*.md` | 新建 / 编辑 | 遵守 §6 写作规范；前沿研究类内容 |
| `theory/**/*_deep_dive.md` | **Pulsar 自动写入** | 见 §6b Deep Dive 分配规则；同 slug 同天允许覆盖；不得手动删除 |
| `playbooks/tools/*.md` | 新建 / 编辑 | 保持 playbook 模板格式 |
| `playbooks/prompts/*.md` | 新建 / 编辑 | 包含示例输入输出 |
| `playbooks/use-cases/*.md` | 新建 / 编辑 | 附真实场景和结果 |
| `playbooks/onboarding/*.md` | 新建 / 编辑 | 面向新成员，步骤清晰 |
| `playbooks/security/**` | 新建 / 编辑 | 安全相关，需注明风险等级 |
| `playbooks/templates/**` | 新建 / 编辑 | 通用模板，注释充分 |
| `scaffolds/` | **仅新建** | 不得修改已有 scaffold，避免破坏下游依赖 |
| `reports/biweekly/` | 新建文件 | 文件名含日期范围；同一天内允许覆盖同名文件 |
| `reports/cross-domain/` | 新建文件 | Pulsar 跨域规则引擎自动写入 |
| `memory/` | ❌ 禁止 | Pulsar 流水线自动生成，任何手动写入均会被覆盖 |
| `monitoring/` | ❌ 禁止 | 监控配置由运维人员管理，智能体不得修改 |

---

## 3. Commit 消息格式

所有提交必须使用下表前缀，保持日志可机器解析。

| 场景 | 格式 | 示例 |
|------|------|------|
| 每日精选 | `📄 daily pick: YYYY-MM-DD (+N apps)` | `📄 daily pick: 2026-03-02 (+3 apps)` |
| 深潜文章 | `📝 deep dive: {module}/{slug}` | `📝 deep dive: 03-engineering/mcp-tool-use-patterns` |
| 工具注册 | `🔧 app index: add {tool-name}` | `🔧 app index: add Cursor` |
| 双周报告 | `📈 report: YYYY-MM-DD ~ YYYY-MM-DD` | `📈 report: 2026-02-17 ~ 2026-03-02` |
| 理论新增 | `📚 theory: add {module}/{name}` | `📚 theory: add 03-engineering/agent-evals` |
| Playbook 更新 | `🛠️ playbook: update {guide-name}` | `🛠️ playbook: update prompt-chaining` |
| Scaffold 新增 | `🗂️ scaffold: add {template-name}` | `🗂️ scaffold: add agent-eval-template` |

---

## 4. 标注系统

### 重要性标注

| 标注 | 含义 | 使用场景 |
|------|------|---------|
| ⚡ | 战略级 — 影响方向、架构选择、行业范式 | 重大模型发布、范式转变、顶级论文 |
| 🔧 | 可操作 — 有实际工程价值，可立即应用 | 新工具、API 变更、最佳实践更新 |
| 📖 | 参考 — 背景知识，暂不需要行动 | 综述、历史回顾、补充读物 |
| ❌ | 不收录 — 噪音、重复、无关信息 | 营销内容、低质转载 |

### 方向标注

| 标注 | 含义 |
|------|------|
| 🎯 | 主方向 — 当前核心跟踪领域 |
| [方向名] | 专项追踪团队标签（如 `[coding-agent]`、`[multimodal]`） |
| ✍️ | 手动添加 — 非流水线自动录入，人工判断 |

---

## 5. landscape/app-index.md 追加格式

每个新工具条目严格遵循以下格式，**追加在文件末尾**：

```
| [Tool Name](URL) | ⚡/🔧/📖 | 🎯/[方向名] | 一句话简介 | YYYY-MM-DD |
```

示例：

```
| [Cursor](https://cursor.com) | ⚡ | 🎯 | 意图驱动的 AI 编程 IDE，Karpathy 推荐范式代表 | 2026-03-02 |
| [Lovable](https://lovable.dev) | 🔧 | [vibe-coding] | 自然语言生成全栈 Web App，零代码门槛 | 2026-03-02 |
```

### 铁律（违反将导致数据损坏）

- **只追加，不删除** — 已录入工具即使停服也保留条目，加注 `[停服 YYYY-MM-DD]`
- **不重排** — 顺序反映录入时间，是历史证据
- **不批量重写** — 每次修改最多新增一批，不得替换现有行
- **格式一致** — 列数、分隔符、标注符号必须与现有条目完全一致

---

## 6. theory/ 写作规范

适用于 `theory/01-principles/` 至 `theory/06-frontier/` 下的新文件。

### 模块选择

新增文章前，根据内容类型选择目标模块：

| 内容类型 | 目标模块 | 子轨道 |
|---------|---------|--------|
| 模型底层机制、推理逻辑、数学原理、心智模型 | `theory/01-principles/` | — |
| Agent 记忆、角色设计、工具接入、多 Agent 协作、Agent UI/API 设计 | `theory/02-agent-design/` | — |
| 护欄设计、失效分类、信任层级 | `theory/03-engineering/` | **支柱 A** |
| Context 注入、Spec 驱动、委派设计、控制平面 | `theory/03-engineering/` | **支柱 B** |
| Eval 体系构建、Eval 持续运维、迭代框架 | `theory/03-engineering/` | **支柱 C** |
| 开发范式、行业转变、新的工作方式 | `theory/04-paradigm/` | — |
| 个人定位、商业变现、职业路径、新组织角色 | `theory/05-strategy/` | — |
| 前沿论文、尚未工程化的突破性研究 | `theory/06-frontier/` | — |

### 文件头部

```markdown
# 中文标题 (English Title)

> **来源**: [论文/文章标题](URL)
> **日期**: YYYY-MM-DD
> **定位**: 一句话说明该框架/架构解决什么问题
```

### 强制开场：X-Ray 开场（2-3 句）

每篇 theory 文档必须以 **X-Ray 开场** 作为正文第一段，回答三个问题：

1. 它在解决什么问题？
2. 它发现/提出了什么？
3. 为什么对 AI 应用工程师重要？

### 推荐 Section 结构

| Section | 说明 | 是否必须 |
|---------|------|---------|
| 系统对比表 | 与同类方案横向对比（至少 3 列） | 推荐 |
| 核心原理 | 关键算法或设计原则，可用伪代码 | 视内容 |
| 工程实战 | 具体实现步骤、配置示例、踩坑记录 | 强烈推荐 |
| 失效模式 | 在哪些场景下该方案会失效 | 推荐 |
| 相关项目 | 引用 `landscape/app-index.md` 中的工具 | 推荐 |

### 语言规范

- **主语言**：中文；专有名词保留英文原文
- **引用**：每篇至少一个外部链接，指向原始论文或官方文档
- **长度**：超过 800 行时考虑拆分子文件

---

## 6b. Deep Dive 分配规则（Pulsar 自动写入）

Pulsar 流水线每周二/四/六从当日 ⚡/🔧 信号中自动生成深潜文章，文件写入对应 theory 模块（不再集中到单一目录）。

### 命名规则

```
theory/{module}/{slug}_deep_dive.md
```

示例：`theory/03-engineering/mcp-tool-use-patterns_deep_dive.md`

### 话题 → 模块分配表

| 话题类型 | 目标模块 | 判断关键词 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sou350121/Agent-Playbook](https://github.com/sou350121/Agent-Playbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
