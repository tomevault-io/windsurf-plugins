---
trigger: always_on
description: 本文档定义了 OpenWebUI 插件开发的标准规范和最佳实践。Copilot 在生成代码或文档时应遵循这些准则。
---

# Copilot Instructions for openwebui-extensions

本文档定义了 OpenWebUI 插件开发的标准规范和最佳实践。Copilot 在生成代码或文档时应遵循这些准则。

This document defines the standard conventions and best practices for OpenWebUI plugin development. Copilot should follow these guidelines when generating code or documentation.

---

## 🏗️ 项目结构与命名 (Project Structure & Naming)

### 1. 语言与代码规范 (Language & Code Requirements)

#### 插件代码 (Plugin Code)

每个插件**必须**采用单文件国际化 (i18n) 设计。严禁为不同语言创建独立的源代码文件（如 `_cn.py`）。

1.  **单代码文件**: `plugins/{type}/{name}/{name}.py`
2.  **内置 i18n**: 必须在代码中根据前端传来的用户语言（如 `__user__` 中的 `language` 或通过 `get_user_language` 脚本读取）动态切换界面显示、提示词和状态日志。

示例目录结构：
```
plugins/actions/export_to_docx/
├── export_to_word.py      # 单个代码文件，内置多语言支持
├── README.md               # 英文文档 (English documentation)
└── README_CN.md            # 中文文档
```

#### 文档 (Documentation)

尽管代码是合一的，但为了市场展示和 SEO，每个插件目录仍**必须**包含双语 README 文件：

- `README.md` - English documentation
- `README_CN.md` - 中文文档

#### 文档交付与审阅 (Documentation Delivery for Review)

当任务涉及文档类内容时，例如 README、Guide、Post、Release Notes、Announcement、Development Docs：

- **必须**同时提供英文版与中文版，方便审阅与校对。
- 若仓库最终只提交英文文件，也**必须**在对话中额外提供中文版草稿给维护者 review。
- 若用户未明确指定只保留单语文件，默认按双语交付处理。
- 中文版的目标是**便于审阅**，应忠实对应英文原意，可在表达上自然调整，但不得遗漏风险、限制、步骤或结论。

#### README 结构规范 (README Structure Standard)

所有插件 README 必须遵循以下统一结构顺序：

1.  **标题 (Title)**: 插件名称，带 Emoji 图标
2.  **元数据 (Metadata)**: 作者、版本、项目链接 (一行显示)
    - 格式: `**Author:** [Fu-Jie](https://github.com/Fu-Jie) | **Version:** x.x.x | **Project:** [OpenWebUI Extensions](https://github.com/Fu-Jie/openwebui-extensions)`
    - **注意**: Author 和 Project 为固定值，仅需更新 Version 版本号
3.  **描述 (Description)**: 一句话功能介绍
4.  **最新更新 (What's New)**: **必须**放在描述之后，仅展示**最近 1 次**更新
5.  **核心特性 (Key Features)**: 使用 Emoji + 粗体标题 + 描述格式
6.  **使用方法 (How to Use)**: 按步骤说明
7.  **配置参数 (Configuration/Valves)**: 使用表格格式，包含参数名、默认值、描述
8.  **支持 (Support)**: **必须**包含，放在配置参数之后、故障排除之前
    - English: `If this plugin has been useful, a star on [OpenWebUI Extensions](https://github.com/Fu-Jie/openwebui-extensions) is a big motivation for me. Thank you for the support.`
    - 中文: `如果这个插件对你有帮助，欢迎到 [OpenWebUI Extensions](https://github.com/Fu-Jie/openwebui-extensions) 点个 Star，这将是我持续改进的动力，感谢支持。`
9.  **其他 (Others)**: 支持的模板类型、语法示例、故障排除等
    - **Changelog**: 统一指向 GitHub 项目历史，不在 README 中列出具体变更

### 2. 插件目录结构 (Plugin Directory Structure)

```
plugins/
├── actions/           # Action 插件 (用户触发的功能)
│   ├── my_action/
│   │   ├── my_action.py          # 单文件，内置 i18n
│   │   ├── README.md              # English documentation
│   │   └── README_CN.md           # Chinese documentation
│   ├── ACTION_PLUGIN_TEMPLATE.py      # 通用 i18n 模板
│   └── README.md
├── filters/           # Filter 插件 (输入处理)
│   └── ...
├── pipes/             # Pipe 插件 (输出处理)
│   └── ...
├── pipelines/         # Pipeline 插件
    └── ...
├── debug/             # 调试与开发工具 (Debug & Development Tools)
│   ├── my_debug_tool/
│   │   ├── debug_script.py
│   │   └── notes.md
│   └── ...
```

#### 调试目录规范 (Debug Directory Standards)

`plugins/debug/` 目录用于存放调试用的脚本、临时验证代码或开发笔记。

**目录结构 (Directory Structure)**:
应根据调试工具所属的插件或功能模块进行子目录分类，而非将文件散落在根目录。

```
plugins/debug/
├── my_plugin_name/      # 特定插件的调试文件 (Debug files for specific plugin)
│   ├── debug_script.py
│   └── guides/
├── common_tools/        # 通用调试工具 (General debug tools)
│   └── ...
└── ...
```

**规范说明 (Guidelines)**:
- **不强制要求 README**: 该目录下的子项目不需要包含 `README.md`。
- **发布豁免**: 该目录下的内容**绝不会**被发布脚本处理。
- **内容灵活性**: 可以包含 Python 脚本、Markdown 文档、JSON 数据等。
- **分类存放**: 任何调试产物（如 `test_*.py`, `inspect_*.py`）都不应直接存放在项目根目录，必须移动到此目录下相应的子文件夹中。

### 3. 文档字符串规范 (Docstring Standard)

每个插件文件必须以标准化的文档字符串开头：

```python
"""
title: 插件名称 (Plugin Name)
author: Fu-Jie
author_url: https://github.com/Fu-Jie/openwebui-extensions
funding_url: https://github.com/open-webui
version: 0.1.0
icon_url: data:image/svg+xml;base64,<base64-encoded-svg>
requirements: dependency1==1.0.0, dependency2>=2.0.0
description: 插件功能的简短描述。Brief description of plugin functionality.
"""
```

#### 字段说明 (Field Descriptions)

| 字段 (Field) | 说明 (Description) | 示例 (Example) |
|--------------|---------------------|----------------|
| `title` | 插件显示名称 | `Export to Word` / `导出为 Word` |
| `author` | 作者名称 | `Fu-Jie` |
| `author_url` | 作者主页链接 | `https://github.com/Fu-Jie/openwebui-extensions` |
| `funding_url` | 赞助/项目链接 | `https://github.com/open-webui` |
| `version` | 语义化版本号 | `0.1.0`, `1.2.3` |
| `icon_url` | 图标 (Base64 编码的 SVG) | 仅 Action 插件**必须**提供。其他类型可选。 |
| `requirements` | 额外依赖 (仅 OpenWebUI 环境未安装的) | `python-docx==1.1.2` |
| `description` | 功能描述 | `将对话导出为 Word 文档` |

#### 图标规范 (Icon Guidelines)

- 图标来源：从 [Lucide Icons](https://lucide.dev/icons/) 获取符合插件功能的图标
- 适用范围：Action 插件**必须**提供，其他插件可选
- 格式：Base64 编码的 SVG
- 获取方法：从 Lucide 下载 SVG，然后使用 Base64 编码
- 示例格式：
```
icon_url: data:image/svg+xml;base64,PHN2ZyB4bWxucz0i...（完整的 Base64 编码字符串）
```

### 4. 依赖管理 (Dependencies)

#### requirements 字段规则

- 仅列出 OpenWebUI 环境中**未安装**的依赖
- 使用精确版本号
- 多个依赖用逗号分隔

```python
"""
requirements: python-docx==1.1.2, openpyxl==3.1.2
"""
```

常见 OpenWebUI 已安装依赖（无需在 requirements 中声明）：
- `pydantic`
- `fastapi`
- `logging`
- `re`, `json`, `datetime`, `io`, `base64`

---

## 💻 核心开发规范 (Core Development Standards)

### 1. Valves 配置规范 (Valves Configuration)

使用 Pydantic BaseModel 定义可配置参数：

```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Fu-Jie/openwebui-extensions](https://github.com/Fu-Jie/openwebui-extensions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
