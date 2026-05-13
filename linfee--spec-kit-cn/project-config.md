---
trigger: always_on
description: > **项目性质**: GitHub Spec Kit 的官方中文复刻版本, 仅做中文本地化, 不开发新特性
---

# Spec Kit CN

> **项目性质**: GitHub Spec Kit 的官方中文复刻版本, 仅做中文本地化, 不开发新特性

## 项目标识

### 核心信息
- **项目名称**: Spec Kit CN
- **原版项目**: [github/spec-kit](https://github.com/github/spec-kit)
- **当前项目**: [linfee/spec-kit-cn](https://github.com/linfee/spec-kit-cn)
- **包名**: `specify-cn`(原版: `specify-cli` 不做更改)
- **命令**: `specify-cn`(原版: `specify`)
- **文档语言**: 中文(原版: 英文)

### 核心原则
1. **功能对等**: 与原版保持完全一致的功能, 不添加新特性
2. **仅做本地化**: 专注于中文翻译和本地化适配
3. **同步优先**: 定期与原版同步, 保持技术更新

### 关键差异
| 项目     | 原版            | 中文版           |
| -------- | --------------- | ---------------- |
| 包名     | `specify-cli`   | `specify-cn-cli` |
| 命令     | `specify`       | `specify-cn`     |
| 文档     | 英文            | 中文             |
| 功能     | 持续开发        | 仅做本地化       |
| 斜杠命令 | `/speckit.plan` | 保持一致         |

---

## 快速参考

### 常用命令
```bash
# 开发环境
uv sync                              # 同步依赖
uv run specify-cn --help             # 运行 CLI
uv run specify-cn check              # 检查工具链

# 测试功能
specify-cn init test-project --ai claude    # 测试项目初始化
specify-cn --help | grep -E "中文|Spec Kit CN"  # 验证中文输出
```

### 自动化翻译工作流
```bash
# 一键自动化翻译(推荐)
/translation-auto      # 全自动翻译流程

# 原版更新处理
/translation-sync      # 智能同步原版更新

# 质量管理
/translation-qa        # 质量保证检查
/translation-fix       # 智能修复问题

# 人工审核
/translation-review    # 人工审核(已存在)
/translation-workflow  # 工作流指南
```

### 项目目录结构
```
项目根目录/
├── src/specify_cli/           # 核心代码(必须同步)
├── templates/                 # 模板文件(需要本地化)
├── scripts/                   # 构建脚本(完全同步, 不翻译)
├── .devcontainer/             # 开发容器配置(完全同步, 不翻译)
├── .github/                   # CI配置(谨慎同步, 不翻译)
├── docs/                     # 项目文档(需要本地化)
├── memory/                    # 项目章程(需要本地化)
├── spec-kit/                 # 原版项目(.gitignore)
├── .claude/commands/         # 翻译自动化命令
├── TERMINOLOGY.md            # 术语表
├── TRANSLATION_STANDARDS.md  # 翻译标准
├── CHANGELOG.md              # 版本记录(独立维护)
└── CLAUDE.md                 # 项目记忆文件
```

### 文件分类与处理策略
| 类别       | 目录/文件                     | 同步策略 | 本地化策略          |
| ---------- | ----------------------------- | -------- | ------------------- |
| 核心代码   | `src/specify_cli/`            | 必须同步 | CLI输出信息需要中文 |
| 模板系统   | `templates/`                  | 结构同步 | 完全中文翻译        |
| 构建脚本   | `scripts/`                    | 完全同步 | 不翻译              |
| 开发环境   | `.devcontainer/`              | 完全同步 | 不翻译              |
| CI配置     | `.github/`                    | 谨慎同步 | 不翻译              |
| 项目文档   | `docs/`, `README.md`          | 结构参考 | 完全中文翻译        |
| 项目章程   | `memory/constitution.md`      | 结构同步 | 完全中文翻译        |
| 原版追踪   | `spec-kit/`                   | 不提交   | 不适用              |
| Agent入口  | `AGENTS.md`                   | 独立维护 | 引用CLAUDE.md       |

---

## 技术架构

### 核心组件

#### Specify CLI 结构 (`src/specify_cli/__init__.py`)
**主要类和函数**: 
- `StepTracker` - 分层步骤进度跟踪 UI 组件
- `select_with_arrows()` - 交互式箭头键选择界面
- `download_template_from_github()` - GitHub releases 模板下载
- `download_and_extract_template()` - 模板下载和解压
- `init()` - 主要项目初始化命令
- `check()` - 工具可用性检查

**关键特性**: 
- 支持多种 AI 编码助手
- 实时进度跟踪和树形显示
- 跨平台支持(Linux/macOS/Windows)
- 自动脚本权限设置(POSIX)
- Git 仓库自动初始化

### 同步策略
**核心策略**: 采用"**核心同步, 界面本地化**"的策略, 确保与原版功能完全对等的同时, 为中文用户提供友好的母语界面.

#### 必须同步的内容
- ✅ **所有类和函数名称**
- ✅ **方法签名和参数**: 完全与原版一致, 确保功能对等
- ✅ **核心算法逻辑**: 模板下载, 解压, Git初始化等核心流程
- ✅ **依赖库和版本**: typer, rich, httpx 等依赖保持同步
- ✅ **AI助手支持**: 所有AI助手的支持逻辑完全一致
- ✅ **构建配置**: hatchling 构建系统保持同步

#### 需要本地化的内容
- 📝 **品牌标识**: 包名, 命令名, GitHub仓库, 横幅标语
- 📝 **用户界面文本**: 错误消息, 状态提示, 交互界面
- 📝 **帮助文档**: 使用说明, 操作指导, 调试信息
- 📝 **输出信息**: CLI 输出, 进度显示, 工具检查结果

### AI 助手支持
| 助手           | CLI 工具       | 目录格式               | 命令格式 | 类型   |
| -------------- | -------------- | ---------------------- | -------- | ------ |
| Claude Code    | `claude`       | `.claude/commands/`    | Markdown | CLI    |
| Gemini CLI     | `gemini`       | `.gemini/commands/`    | TOML     | CLI    |
| GitHub Copilot | 无(IDE 集成) | `.github/prompts/`     | Markdown | IDE    |
| Cursor         | `cursor-agent` | `.cursor/commands/`    | Markdown | CLI    |
| Qwen Code      | `qwen`         | `.qwen/commands/`      | TOML     | CLI    |
| opencode       | `opencode`     | `.opencode/command/`   | Markdown | CLI    |
| Windsurf       | 无(IDE 集成) | `.windsurf/workflows/` | Markdown | IDE    |
| Codex          | `codex`        | `.codex/`              | Markdown | CLI    |
| Kilocode       | `kilocode`     | `.kilocode/`           | Markdown | CLI    |
| Auggie         | `auggie`       | `.auggie/`             | Markdown | CLI    |
| Amazon Q Developer CLI | `q` | `.amazonq/prompts/` | Markdown | CLI    |

---

## 开发环境配置 (.devcontainer/)

### Devcontainer 概述
`.devcontainer/` 目录是 v0.0.78 新增的开发容器配置, 提供完整的开发环境自动化设置.

### 配置文件结构
```
.devcontainer/
├── devcontainer.json     # 主配置文件, 定义容器环境和工具
└── post-create.sh       # 容器创建后自动执行脚本
```

### 核心功能
- **预配置开发环境**: Python 3.13 + uv 包管理器
- **AI助手自动安装**: 自动安装所有支持的AI编码助手
- **VS Code集成**: 预装必要的扩展和设置
- **多语言支持**: Node.js, .NET, Git 等开发工具
- **端口转发**: 8080端口用于文档站点预览

### 包含的AI助手
- GitHub Copilot CLI
- Claude Code CLI
- Codex CLI
- Gemini CLI
- Auggie CLI
- Qwen Code CLI
- OpenCode CLI
- Amazon Q Developer CLI
- CodeBuddy CLI

### 使用方式
1. 在 VS Code 中打开项目
2. 提示"在容器中重新打开"时选择确定
3. 等待容器构建和脚本执行完成
4. 所有AI助手将自动安装并可用

### 同步策略
- **完全同步**: 与原版保持100%一致
- **不翻译**: 所有配置文件保持英文

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Linfee/spec-kit-cn](https://github.com/Linfee/spec-kit-cn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
