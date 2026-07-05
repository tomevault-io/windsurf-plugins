---
trigger: always_on
description: 本文件为 AI Agent 提供项目操作手册与约束清单，确保 Agent 行为可控、可复现。
---

# Repository Guidelines

本文件为 AI Agent 提供项目操作手册与约束清单，确保 Agent 行为可控、可复现。

---

## 1. Mission & Scope（目标与边界）

### 允许的操作
- 读取、修改顶层文档：`README.md`、`AGENTS.md`、`CONTRIBUTING.md` 等
- 读取、修改 `docs/`、`prompts/`、`skills/`、`tools/config/`、`tools/external/` 下的文档与代码
- 执行 `make lint`、`make check-links`、`make check-details`、`make check-doc-structure`、`make check-directory-docs`、`make check-metadata`、`make check-ai-citation`、`make sync-doc-toc`、prompts-library 转换工具
- 新增/修改提示词、技能、文档
- 提交符合规范的 commit

### 禁止的操作
- 修改 `.github/workflows/` 中的 CI 配置（除非任务明确要求）
- 修改 `LICENSE`、`CODE_OF_CONDUCT.md`
- 在代码中硬编码密钥、Token 或敏感凭证
- 未经确认的大范围重构

### 敏感区域（禁止自动修改）
- `.github/workflows/*.yml` - CI/CD 配置
- `.env*` 文件（如存在）

---

## 2. Golden Path（推荐执行路径）

```bash
# 1. 拉取最新代码
git pull --rebase origin develop

# 2. 初始化外部仓库指针
git submodule update --init --recursive

# 3. 运行 lint 检查
make lint

# 4. 执行修改任务
# ...

# 5. 再次 lint 验证
make lint

# 6. 提交变更
git add -A
git commit -m "feat|fix|docs|chore: scope - summary"
git push origin develop
```

---

## 3. Must-Run Commands（必须执行的命令清单）

### 环境要求
- Node.js 22+（通过 `npx --yes markdownlint-cli@0.48.0` 运行固定版本 Markdown lint）
- Python 3.8+（用于 prompts-library 工具与链接检查脚本）
- Git

### 核心命令

| 命令 | 用途 | 前置条件 |
|:---|:---|:---|
| `make help` | 列出所有 Make 目标 | 无 |
| `make lint` | 校验全仓库 Markdown | Node.js 22+；通过 `npx --yes markdownlint-cli@0.48.0` 执行 |
| `make check-links` | 校验仓库内 Markdown 相对链接 | Python 3 |
| `make check-details` | 校验 Markdown 折叠块 `<details>/<summary>` 结构 | Python 3 |
| `make check-doc-structure` | 校验 docs 线性 README 标准块顺序、主章节顺序、重复锚点与目录入口 | Python 3 |
| `make check-directory-docs` | 校验仓库自有目录 README/AGENTS 覆盖 | Python 3 |
| `make check-metadata` | 校验 metadata 路径与锚点 | Python 3 |
| `make check-ai-citation` | 校验 llms 与 AI 引用语料路径和锚点 | Python 3 |
| `make sync-doc-toc` | 根据 taxonomy 和文档锚点重建 docs 细粒度目录 | Python 3 |
| `make test` | 执行本地质量门禁 | Node.js 22+、Python 3 |
| `git submodule update --init --recursive` | 初始化外部 Git 仓库指针 | Git |
| `cd tools/prompts-library && python3 main.py` | 提示词格式转换 | `pip install -r tools/prompts-library/requirements.txt` |

### Python 依赖来源
- prompts-library 主入口依赖：`tools/prompts-library/requirements.txt`
- prompts-library Google API / JSONL 辅助脚本依赖：`tools/prompts-library/scripts/requirements.txt`

### prompts-library 支持的转换模式
1. Excel → Docs：将 Excel 工作簿转换为 Markdown 文档目录
2. Docs → Excel：将 Markdown 文档目录还原为 Excel 工作簿
3. Docs → JSONL：将 Markdown 文档转换为 JSONL 格式
4. JSONL → Excel：将 JSONL 转换为 Excel
5. Excel(JSONL) → JSONL：将内部 JSONL 格式的 Excel 转换为 JSONL 目录（每个工作表一个 JSONL 文件）

---

## 4. Code Change Rules（修改约束）

### 架构原则
- 保持根目录扁平，避免巨石文件
- 三层内容架构：`docs/` (知识) → `prompts/` (指令) → `skills/` (能力)

### 模块边界
- `docs/` - 中文知识库（方法论/入门/实战/资源）
- `prompts/` - 提示词入口与云端索引
- `skills/` - 可复用技能库（每个子目录一个 Skill）
- `tools/config/` - 工具与开发配置（例如 Codex CLI）
- `tools/external/` - 外部工具与依赖（含 Git submodule）

### 依赖添加规则
- 新增工具或库时记录安装方式、最小版本与来源
- 外部依赖来源记录在 `tools/external/` 目录下
- 引入第三方脚本需标明许可证与来源

### 禁止行为
- 禁止"顺手重构/大范围改动"除非任务明确要求
- 禁止删除现有测试用例（除非任务要求）
- 禁止在代码中硬编码敏感信息

---

## 5. Style & Quality（风格与质量标准）

### 格式化工具
- Markdown：`Makefile` 固定调用 `markdownlint-cli@0.48.0`（通过 `make lint` 执行）
- CI 自动检查：`.github/workflows/ci.yml`

### 命名约定
- 文档、注释、日志使用中文
- 代码符号统一英文且语义直白
- 文件名小写加中划线或下划线

### 缩进与排版
- 全仓保持空格缩进（2 或 4 空格不混用）
- 行宽控制在 120 列内

### 设计品味
- 优先消除分支与重复
- 函数单一职责且短小

---

## 6. Project Map（项目结构速览）

```
.
├── README.md                    # 项目主文档
├── AGENTS.md                    # AI Agent 行为准则（本文件）
├── llms.txt                     # 面向 AI 助手的短上下文入口
├── Makefile                     # 自动化脚本
├── LICENSE                      # MIT 许可证
├── CODE_OF_CONDUCT.md           # 行为准则
├── CONTRIBUTING.md              # 贡献指南
├── .gitattributes               # GitHub Linguist 语言统计规则
├── .gitignore                   # Git 忽略规则
│
├── docs/                        # 核心知识库
│   ├── README.md                # docs 总索引
│   ├── getting-started/         # 从零开始、学习地图、环境与 AI CLI 配置
│   ├── concepts/                # 核心概念、方法论与工程思想
│   ├── philosophy/              # 哲学方法论、思维模型与底层认知模型
│   ├── references/              # 清单、约束、常见坑、模板和技术栈参考
│   ├── research/                # 新技术、优秀 repo 与工程范式研究
│   └── workflow/                # 开发流程、质量门禁和交付闭环
│
├── prompts/                     # 提示词库入口（指向云端表格）
│   ├── README.md                # 在线表格链接
│   └── AGENTS.md                # prompts/ 目录规则
│
├── skills/                      # 技能库（每个子目录一个 Skill）
│   ├── README.md                # skills 总览与索引
│   ├── AGENTS.md                # skills/ 目录规则
│   ├── auto-skill/              # 元技能核心
│   └── claude-official-skills/  # Claude 官方 skills 软链接入口
│
├── assets/                      # 静态资产与外部资源入口
│   ├── README.md                # 外部资源在线表格入口
│   ├── AGENTS.md                # assets/ 目录规则
│   ├── ai-citation/             # AI 引用语料包与 llms-full
│   ├── images/                  # 图片资产
│   ├── templates/               # 模板附件
│   └── datasets/                # 示例数据或数据说明
│
├── scripts/                     # 自动化脚本
│   ├── README.md                # scripts 目录说明
│   ├── AGENTS.md                # scripts 目录规则
│   └── check-local-links.py     # Markdown 相对链接检查
│
├── tools/                       # 工具、本地配置与外部仓库
│   ├── README.md                # tools 目录说明
│   ├── AGENTS.md                # tools 目录规则
│   ├── config/                  # 工具与开发配置（含 Codex CLI）
│   ├── prompts-library/         # Excel ↔ Markdown 互转工具

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ziiyuecheng/vibe-coding-cn](https://github.com/ziiyuecheng/vibe-coding-cn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
