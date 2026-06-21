---
trigger: always_on
description: > **文档定位**:本文档是 **AI 代码助手(Codex/Claude/GitHub Copilot)** 的操作手册,定义了自动化贡献的强制规则与质量标准。人工贡献者请参考 [docs/contributing/](docs/contributing/)。
---

# Multiple Personality System Wiki - AI 代理工作规范

> **文档定位**:本文档是 **AI 代码助手(Codex/Claude/GitHub Copilot)** 的操作手册,定义了自动化贡献的强制规则与质量标准。人工贡献者请参考 [docs/contributing/](docs/contributing/)。

!!! danger "AI 代理必读 - 违反 = 任务失败"
    本文档优先级 **高于** AI 默认行为。所有自动化操作必须严格遵循以下规范。

---

## 🎯 快速开始(AI 代理 30 秒速查)

### 核心原则

```yaml
✅ 必须遵守:
  - 语言: 简体中文(代码/命令除外)
  - 路径: 相对路径(禁止绝对路径)
  - 提交: 小步提交 + Conventional Commits
  - 时间戳: 让 CI 自动更新(勿手动修改)
  - Guide 同步: 修改词条必须更新对应 Guide

❌ 严禁操作:
  - 使用绝对路径链接(如 /docs/entries/DID.md)
  - 手动修改 updated 时间戳(CI 自动处理)
  - 破坏 Frontmatter 结构
  - 跳过链接检查
  - 在 docs/entries/ 创建子目录
```

### 决策树(任务开始前必查)

```text
┌─ 修改词条?
│  ├─ 读取 docs/TEMPLATE_ENTRY.md
│  ├─ 检查 Frontmatter(title/topic/tags,勿碰 updated)
│  ├─ 更新对应 Guide(见 §5 映射表)
│  └─ 使用相对路径(词条间直接 DID.md,跨目录 ../entries/DID.md)
│
┌─ 开发/修改工具?
│  ├─ 修改 tools/*.py
│  ├─ 同步更新 docs/dev/Tools-Index.md
│  └─ 优先使用 make；底层 Python 命令仍使用 uv run python3(不是裸 python3)
│
┌─ 大规模重构?
│  ├─ 先列影响范围
│  ├─ 检查 7 个 Guide 是否需更新
│  ├─ 小步提交 + 回滚指引
│  └─ PR 说明自动化方法(正则/脚本/范围)
│
└─ 提交前检查
   ├─ make check
   ├─ make build
   └─ uv run mkdocs build --strict(额外严格检查,可选)
```

---

## 📑 目录导览

### 🔴 强制阅读(执行前必看)

- [§2 AI 代理强制规则](#2-ai-代理强制规则) ⚠️ 最高优先级
- [§3 文件结构与路径](#3-文件结构与路径)
- [§4 Frontmatter 规范](#4-frontmatter-规范)
- [§5 链接规范与 Guide 映射](#5-链接规范与-guide-映射)
- [§6 提交与 CI 流程](#6-提交与-ci-流程)

### 🟡 按需查阅

- [§7 站点配置](#7-站点配置)
- [§8 工具开发](#8-工具开发)
- [§9 Python 环境](#9-python-环境)
- [§10 标签规范](#10-标签规范-v20)
- [§11 常见问题](#11-常见问题)

---

## 2. AI 代理强制规则

!!! danger "违反以下任一规则 = 任务失败"

### 2.1 语言规范

| 规则 | 说明 | 示例 |
|------|------|------|
| ✅ 简体中文 | 所有文本内容、提交信息 | `feat: 新增 Grounding 词条` |
| ✅ 一级标题 | `中文名(English/缩写)` | `# 解离性身份障碍(DID)` |
| ⚠️ 诊断类词条 | 括号内必须用标准缩写 | `解离性身份障碍(DID)` 不是 `解离性身份障碍` |

### 2.2 路径规范(高频错误)

| 场景 | ✅ 正确 | ❌ 错误 |
|------|---------|---------|
| 词条间链接 | `[DID](DID.md)` | `[DID](/docs/entries/DID.md)` |
| 词条→其他目录 | `[贡献指南](../contributing/index.md)` | `[贡献指南](/contributing/index.md)` |
| 其他目录→词条 | `[DID](../entries/DID.md)` | `[DID](DID.md)` |

### 2.3 提交规范

```text
<type>: <description>

type 必须是:
  feat     新增词条/功能
  fix      修复错误/错别字
  docs     文档说明调整
  refactor 结构调整/重构
  chore    构建/配置/依赖
  style    格式化(非语义)

示例:
  feat: 新增 Grounding 技巧词条
  fix: 修正 DID 诊断标准引用
  docs: 更新贡献指南链接规范
```

### 2.4 自动化操作规范

| 操作 | 规范 |
|------|------|
| ✅ 小步提交 | 每次提交最小可审查单位 |
| ✅ 提交前检查 | 必须运行 `check_links.py` + `check_tags.py` |
| ✅ PR 说明 | 大规模自动化需注明方法(正则/脚本名/范围) |
| ✅ 工具同步 | 修改 `tools/` 必须更新 `docs/dev/Tools-Index.md` |
| ❌ 无迹可查 | 禁止无法验证来源的批量修改 |
| ❌ 破坏索引 | 禁止破坏导航/引用完整性 |
| ⚠️ 大规模重构 | 必须附回滚指引 |

---

## 3. 文件结构与路径

### 3.1 目录结构(只读规则)

```text
docs/
├── entries/              # 词条存放(禁止子目录)
│   ├── DID.md
│   └── Grounding.md
├── contributing/         # 贡献指南(拆分多文件)
├── dev/
│   └── Tools-Index.md   # 工具文档(修改 tools/ 必须同步)
├── assets/
│   ├── figures/         # 流程图/示意图
│   ├── images/          # 封面/截图
│   └── icons/           # 图标
├── index.md             # 首页
├── README.md
├── Glossary.md
└── TEMPLATE_ENTRY.md    # 词条模板(必读)

tools/                    # 脚本与工具
├── check_links.py       # 链接检查(提交前必跑)
├── check_tags.py        # 标签验证(提交前必跑)
├── fix_markdown.py      # 格式修复(CI 自动)
└── update_git_timestamps.py  # 时间戳(CI 自动)
```

### 3.2 关键约束

!!! danger "严格遵守"

    - ❌ **禁止**在 `docs/entries/` 创建子目录(分类通过 Frontmatter tags 管理)
    - ✅ **必须**将静态资源放在 `docs/assets/` 对应子目录
    - ✅ **必须**修改 `tools/` 后同步更新 `docs/dev/Tools-Index.md`

---

## 4. Frontmatter 规范

### 4.1 必需字段

```yaml
---
title: 词条标题              # 必需
topic: 所属主题              # 必需,见下方主题列表
tags:                       # 必需,至少 1 个,最多 5 个
  - dx:解离性身份障碍（DID）                  # 格式: prefix:名称
  - sx:切换(Switch)
updated: YYYY-MM-DD        # 必需,但 CI 自动维护,勿手动改
---
```

### 4.2 可选字段

```yaml
search:
  boost: 1.8               # 搜索权重(仅核心词条使用)
```

**权重分级参考**:

| 优先级 | 值 | 适用 | 示例 |
|--------|-----|------|------|
| 最高 | 2.0 | 诊断类 | DID/OSDD/PTSD/CPTSD |
| 高 | 1.8 | 核心概念 | Alter/System/Switch/Grounding |
| 中高 | 1.5 | 重要概念 | Protector/Host/Dissociation |
| 默认 | 1.0 | 普通词条 | 无需设置 |

### 4.3 主题列表(topic 必须从此选择)

```text
诊断与临床      # DID/OSDD/CPTSD/焦虑障碍/情绪障碍
系统运作        # 前台切换/共同意识/记忆管理/内部空间
实践指南        # Tulpa 三阶段/冥想/可视化/接地技巧
创伤与疗愈      # 创伤机理/PTSD 症状/三阶段治疗模型
角色与身份      # 宿主/守门人/保护者/照护者
理论与分类      # 结构性解离/依恋理论/自我决定理论
文化与表现      # 影视/文学/动画/游戏主题
```

### 4.4 例外文件(无需 updated 字段)

- `guides/*-Guide.md`(如 `guides/Clinical-Diagnosis-Guide.md`)
- `guides/*-index.md`(如 `guides/Clinical-Diagnosis-index.md`)
- `dev/*-Index.md`(如 `dev/Tools-Index.md`)

---

## 5. 链接规范与 Guide 映射

### 5.1 链接路径速查表

| 链接场景 | 格式 | 示例 |
|---------|------|------|
| **词条间** | `文件名.md` | `[DID](DID.md)` |
| **词条→Guide** | `../guides/XX-Guide.md` | `[诊断指南](../guides/Clinical-Diagnosis-Guide.md)` |
| **Guide→词条** | `../entries/XX.md` | `[DID](../entries/DID.md)` |
| **Guide间** | `文件名.md` | `[系统运作](System-Operations-Guide.md)` |
| **词条→首页** | `../index.md` | `[首页](../index.md)` |

### 5.2 Guide 映射表(修改词条必须同步更新对应 Guide)

| 词条主题(topic) | 对应 Guide 文件 | 操作 |
|----------------|----------------|------|
| **诊断与临床** | `guides/Clinical-Diagnosis-Guide.md` | 新增/修改/删除词条时更新链接和描述 |
| **系统运作** | `guides/System-Operations-Guide.md` | 同上 |
| **实践指南** | `guides/Practice-Guide.md` | 同上 |
| **创伤与疗愈** | `guides/Trauma-Healing-Guide.md` | 同上 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mps-team-cn/Multiple_personality_system_wiki](https://github.com/mps-team-cn/Multiple_personality_system_wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-21 -->
