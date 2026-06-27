---
trigger: always_on
description: > 本文件供 AI Coding Agent 阅读。项目语言为**简体中文**。
---

# AGENTS.md — init-knowledge-base

> 本文件供 AI Coding Agent 阅读。项目语言为**简体中文**。

---

## 项目概述

`init-knowledge-base` 是一个 **Kimi Code CLI Project Skill**，用于一键初始化标准化的 Obsidian 知识库（Vault）项目骨架。

- **灵感来源**：Andrej Karpathy 的 [LLM Wiki 规范](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- **核心理念**：将碎片化信息编译成**结构化、高度相互链接**的知识网络
- **使用场景**：用户在 Kimi Code CLI 中输入 `/init-vault <名称>` 或自然语言"新建知识库"时，Skill 将 `_templates/` 下的所有模板文件复制到目标目录，生成完整的 Obsidian Vault 结构

**注意**：本项目本身不是传统软件项目，没有 package.json、pyproject.toml、Cargo.toml 等构建配置文件，也没有编译或打包步骤。它是一个纯模板与配置仓库，依赖 Markdown、CSS、JSON 和 Kimi Code CLI 的 Skill 系统运行。

---

## 技术栈与运行时架构

| 层级 | 技术 | 说明 |
|------|------|------|
| AI 运行时 | [Kimi Code CLI](https://github.com/moonshot-ai/Kimi-CLI) | Skill 的加载与执行环境 |
| 知识库客户端 | [Obsidian](https://obsidian.md/)（桌面端） | 用户主要的阅读与编辑界面 |
| 数据查询 | Dataview（Obsidian 社区插件） | `wiki/index.md` 动态仪表盘依赖此插件 |
| 模板引擎 | Templater（Obsidian 社区插件） | `templates/` 目录下的 4 个模板由其解析 |
| 样式 | CSS Snippets | `.obsidian/snippets/` 下的 3 个 CSS 文件 |
| 版本控制 | Git | 仅追踪配置与内容，排除插件二进制文件 |

### 运行时架构

```
用户通过 Kimi Code CLI 触发 /init-vault
        ↓
CLI 读取本项目的 SKILL.md
        ↓
按 SKILL.md 定义的"生成流水线"，将 _templates/ 复制到用户指定目录
        ↓
生成的 Vault 包含：Obsidian 配置 + CSS 样式 + 模板 + 5 个 Agent Skills
        ↓
用户在 Obsidian 中打开 Vault，使用 /ingest、/query、/lint、/canvas 命令与 AI 交互
```

---

## 目录结构与模块划分

```
项目根目录/
├── SKILL.md                          # Skill 定义与生成流水线（Kimi Code CLI 入口）
├── README.md                         # 面向人类用户的项目说明
├── LICENSE                           # MIT 许可证
├── AGENTS.md                         # 本文件
└── _templates/                       # 所有模板文件，初始化时复制到目标项目
    ├── README.md                     # 生成的 Vault 的 README（含 {vault_name} 占位符）
    ├── CLAUDE.md                     # Agent 行为契约与规范（原样复制）
    ├── OBSIDIAN_SETUP.md             # Obsidian 配置指南（原样复制）
    ├── .gitignore                    # 忽略规则（原样复制）
    ├── .obsidian/                    # Obsidian 统一配置
    │   ├── app.json                  # 编辑器设置（新文件位置、附件路径、显示选项等）
    │   ├── appearance.json           # 外观 + 启用的 CSS 片段清单
    │   ├── core-plugins.json         # 核心插件开关列表
    │   ├── community-plugins.json    # 推荐社区插件列表（dataview, templater 等）
    │   └── snippets/                 # CSS 样式片段
    │       ├── wiki-reading.css      # 阅读排版优化
    │       ├── wiki-callouts.css     # Callout 语义化颜色 + 左侧边条
    │       └── wiki-components.css   # 双链、外部链接、标签、图片、代码块增强
    ├── templates/                    # Templater 模板文件夹
    │   ├── entity.md                 # 实体模板（人物、公司、工具、产品）
    │   ├── concept.md                # 概念模板（框架、方法论）
    │   ├── source.md                 # 来源摘要模板
    │   └── synthesis.md              # 综合分析模板
    ├── wiki/                         # 知识编译输出层骨架
    │   ├── index.md                  # Dataview 动态仪表盘（全局内容字典）
    │   ├── log.md                    # 操作日志（Append-only）
    │   └── mocs/                     # 主题地图（Map of Contents）
    │       ├── README.md             # MOC 创建指南与 Dataview 查询模板
    │       ├── MOC-技术.md           # 技术领域 MOC
    │       ├── MOC-商业.md           # 商业领域 MOC
    │       ├── MOC-人物.md           # 人物与机构 MOC
    │       └── MOC-待整理.md         # 草稿与待分类 MOC
    └── .claude/                      # Agent Skills
        └── skills/
            ├── ingest/SKILL.md       # 将 raw/ 资料编译到 wiki/
            ├── query/SKILL.md        # 在知识库中搜索与回答
            ├── lint/SKILL.md         # 检查死链、孤儿页面、逻辑冲突
            ├── obsidian-markdown/SKILL.md  # Obsidian Markdown 语法规范
            └── json-canvas/SKILL.md  # Canvas 可视化与知识图谱
```

### 模块说明

| 模块 | 职责 | 修改注意事项 |
|------|------|-------------|
| `SKILL.md` | 定义 Skill 元数据、触发场景、7 步生成流水线 | 修改触发命令或流水线步骤时必须同步更新 |
| `_templates/` | 所有初始化时复制的文件 | 新增/删除/修改模板文件后，必须同步更新 `SKILL.md` 中的"模板位置"和"生成流水线"章节 |
| `_templates/.obsidian/` | Obsidian 应用配置 | JSON 文件需保证语法正确；`community-plugins.json` 中的插件 ID 必须与 Obsidian 社区插件市场一致 |
| `_templates/templates/` | 4 个 Templater 模板 | 模板使用 Templater 语法（如 `<% tp.file.title %>`），修改时需确保语法有效 |
| `_templates/wiki/index.md` | Dataview 仪表盘 | 包含 Dataview 查询代码块，修改查询语法需测试兼容性 |
| `_templates/.claude/skills/` | 5 个子 Skill | 每个 SKILL.md 独立定义一套工作流；修改约束条件时需检查是否与 `CLAUDE.md` 冲突 |

---

## 开发规范与约定

### 语言与编码

- **所有文件使用 UTF-8 编码**。
- **工作语言为简体中文**：所有文档、注释、模板内容、Agent 指令均使用简体中文。
- 生成的 Vault 中的 `CLAUDE.md` 强制规定：Agent 无论收到何种语言输入，都必须使用简体中文思考和回复。

### 命名规范

| 类型 | 规范 | 示例 |
|------|------|------|
| Vault 项目名称 | kebab-case 或 snake_case | `physics-notes`, `study_vault` |
| Wiki 页面 — Entities/Concepts | TitleCase（帕斯卡命名） | `Transformer`, `OpenAI` |
| Wiki 页面 — Sources/Syntheses | kebab-case | `summary-article-name.md` |
| 文件路径 | 小写，连字符分隔 | `wiki/concepts/`, `raw/01-articles/` |

### 文件操作权限（核心架构约束）

- **`raw/`（不可变层）**：绝对只读。存放原始素材。Agent 禁止修改或删除此目录下任何文件。
- **`assets/`（媒体资产层）**：存放图片、PDF。引用语法为 Obsidian 标准 `![[文件名.png]]`。
- **`wiki/`（编译输出层）**：Agent 的工作区。创建、更新、提炼知识并解决矛盾。

### Wiki 页面 Frontmatter（YAML）规范

所有生成的 wiki 页面必须包含以下 YAML 头部：

```yaml
---
title: "页面标题"
type: concept | entity | source | synthesis | moc
aliases: []
tags: [知识标签]
sources: [关联的raw文件相对路径]
created: YYYY-MM-DD
last_updated: YYYY-MM-DD
status: draft | finished | archived
---
```

**类型扩展字段**（根据页面类型可选补充）：
- `entity` → `entity_type`: 人物 / 公司 / 产品 / 工具 / 机构 / 地点 / 其他
- `concept` → `complexity`: 复杂度评分，如 ⭐⭐⭐☆☆
- `source` → `source_type`: 文章 / 论文 / 书籍 / 视频 / 播客 / 会议 / 其他；`credibility`: ⭐⭐⭐☆☆

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lululu811/init-knowledge-base](https://github.com/lululu811/init-knowledge-base) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
