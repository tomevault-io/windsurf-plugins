---
trigger: always_on
description: 在本地 Markdown 工作空间中运行技能优先的知识工作流
---


<objective>
在本地 Markdown 工作空间中运行技能优先的知识工作流。
工作空间跨会话、跨项目、跨来源积累知识——
不是每次查询都从原始文档重新发现，而是编译和维护一个持续增长的持久化 wiki。

wiki 中的每条声明都必须引用来源。每次查询都必须声明答案来源。
每个大型源文档都必须能够分段处理，并支持可中断的进度。

适用场景：文档导入、项目驱动的知识积累、
代码-文档绑定、大文件增量式 wiki 构建。
Obsidian 是可选的界面，而非先决条件。
</objective>

<inputs>
用户输入可以是以下之一：
- 要捕获的 URL
- 工作空间内外的本地文件路径
- 粘贴的文本内容
- 要从 wiki 回答的问题
- 审查、重组、合并、拆分、重命名或刷新 wiki 页面的请求
- 纠正过时或不准确 wiki 内容的请求
- 为现有项目初始化 wiki 结构的请求
- 在源代码和 wiki 页面之间建立双向链接的请求
- 大型文档结构映射或代码库索引的请求
</inputs>

## 使用方式

直接告诉我要做什么：

| 用户意图示例 | 执行的工作流 | 说明 |
|-------------|-------------|------|
| "初始化这个项目的wiki" / "帮我建wiki结构" | `project-init` | 扫描项目，搭建 raw/wiki 骨架 |
| "把这个PDF建档" / "捕获这个URL" | `capture` | 保存到 raw 层，记录元数据 |
| "建档第5章" / "把这份文档消化进wiki" | `ingest` | 提取知识，更新 wiki 页面 |
| "问个问题" / "wiki里怎么说的" | `query` | 意图路由、渐进式加载、声明交叉验证、反馈闭环 |
| "帮我研究一个主题" / "我想学习XX" | `research` | 自动搜索、筛选、下载、导入 |
| "来源更新了" / "检查同步状态" | `sync` | 增量同步、受影响页面定位、过时标记 |

| "检查wiki一致性" / "review一下" | `review` | 9项健康检查，CRITICAL/WARN/INFO |
| "编译验证" / "编译wiki" | `compile` | 三阶段编译检查 — 来源覆盖、声明完整性、矛盾检测 |
| "合并这两个页面" / "整理一下这个目录" | `curate` | 重组结构，保留关联 |
| "帮我绑定代码和手册" | `code-anchor` | 双向绑定：wiki知道代码，代码知道wiki |
| "画出手册结构" / "PDF目录提取" | `map-document` | 提取 PDF/DOCX/PPTX 大纲 |
| "索引这个代码目录" | `index-codebase` | 生成代码符号地图 |

歧义请求默认选择最窄的匹配工作流。

<vault_assumptions>
- 在 Claude 能够读取和更新的本地 Markdown 工作空间中工作。
- 优先使用直接的文件系统读写以确保可靠性。
- 如果 `obsidian` CLI 可用，将其视为搜索和导航的可选增强工具。
- 如果 CLI 不可用，回退到 `Glob` 和 `Grep`。
- 仅当用户明确要求时才使用 `obsidian://open` 或 GUI 打开行为。
</vault_assumptions>

## 配置说明

### Claude Code 设置

建议在 `~/.claude/settings.json` 中配置：

```json
{
  "permissions": {
    "allow": ["Bash(*)", "Read(*)", "Write(*)", "Edit(*)", "Glob(*)", "Grep(*)"],
    "deny": ["Bash(rm -rf:*)", "Bash(chmod:*)"]
  }
}
```

### 工作空间本地配置

在工作空间根目录创建 `.claude/settings.local.json`，可覆盖全局配置：

```json
{
  "permissions": {
    "deny": ["Bash(rm -rf:*)"]
  }
}
```

### 必需依赖

| 工具 | 用途 | 安装 |
|------|------|------|
| `uv` | Python 依赖隔离与动态库安装 | `curl -LsSf https://astral.sh/uv/install.sh \| sh` |

**动态工具发现原则：** 不在 SKILL.md 中硬编码特定 Python 库列表。
Claude Code 根据实际要处理的文件类型，自行判断需要哪些 Python 库，
通过 `uv run --with <library>` 动态安装。`uv` 的 `--with` 机制使得无需预装即可使用任意 PyPI 包。

常见场景参考（非强制列表，Claude Code 自行判断实际需要的库）：

| 文件类型 | 可能需要 |
|---------|---------|
| PDF（有目录） | `PyPDF2` |
| PDF（无目录/扫描件） | `pdfplumber`（或 `PyPDF2` 备用） |
| DOCX | `python-docx` |
| PPTX | `python-pptx` |
| C/C++ 源码 | `tree-sitter-languages` |
| 其他语言源码 | 对应的 tree-sitter 语言包（如 `tree-sitter-python`、`tree-sitter-rust` 等） |

项目自带工具脚本（`sub-skills/tools/*.py`）使用 Python 标准库，无额外依赖，
通过 `uv run python sub-skills/tools/<script>.py` 运行。

### Windows 注意事项

- 始终将文件路径作为命令行参数传递，避免 heredoc 中的 unicode 路径问题

<core_model>
工作流假设三个层次：
1. `raw/` — 捕获的原始来源和附件
2. `wiki/` — 维护的知识页面
3. `CLAUDE.md` + 模板 — 模式定义和工作流规则

wiki 是主要的回答层面。原始来源是可追溯的输入，而非默认的响应层面。
</core_model>

<tool_assumptions>
繁重的工作（PDF 解析、代码索引）委托给外部工具。
`uv` 是唯一的必需依赖，其他 Python 库按需动态安装。

| 工具 | 用途 | 必需 | 后备 |
|------|---------|----------|---------|
| `uv` | Python 依赖隔离与动态库安装 | 是 | 无 — 必须安装 |

**安装 `uv`：** `curl -LsSf https://astral.sh/uv/install.sh | sh`
（Git Bash / Linux / macOS — 三个平台命令相同）

**动态工具发现：** Claude Code 根据要处理的文件类型自行判断需要哪些 Python 库，
通过 `uv run --with <library>` 即时安装使用，无需预装。
每个工作流定义中可给出该场景常用的库建议，但不作为硬性依赖。

**平台说明：**
- 在 Windows 上（Git Bash / MSYS），始终将文件路径作为命令行参数传递，切勿通过 stdin 或 heredoc — 避免空格和 unicode 字符的引号问题
</tool_assumptions>

<path_rule>
始终使用工作空间相对路径注释有意义的文件操作。

示例：
- `→ 写入: raw/sources/article-name.md`
- `← 读取: wiki/concepts/self-attention.md`
- 搜索结果应包含文件路径
- 摘要应列出每个创建或更新的路径

**Windows / 非 ASCII 路径兼容性：**
- 当 Edit 的 old_string 包含非 ASCII 字符（中文、日文等）时，Edit 工具可能静默失败。
  解决方法：使用 `uv run python` 执行替换，或将编辑拆分为避免在 old_string 中使用非 ASCII 字符串的较小片段。
- 始终将文件路径作为命令行参数传递给 Python 脚本，切勿通过 heredoc 或 stdin — 避免 unicode 文件名的引号问题。
- 临时文件位置遵循全局规则：所有临时文件写入 `raw/.tmp/`。
</path_rule>

<global_rules>
- 捕获后保持 `raw/` 不可变。
- 优先更新现有页面，而非创建近似重复。
- 回答问题时先查阅 wiki。
- 显式呈现矛盾和不确定性，而非默默抹平。
- 将人类修正视为一等工作流。
- 对于有意义的捕获、导入、保存、审查、整理和修正操作，更新 `log.md`。
  对于长时间运行或多会话任务，在日志条目末尾包含 `task_status` 块，以便中断的会话可以恢复：
  ```markdown
  ## [2026-04-23] ingest | R7F0C014 ch12 WDT
  - Created wiki/sources/r7f0c014-manual-ch12-wdt.md ✅
  - Updated wiki/peripherals/wdt.md ✅
  - **task_status: s12 (complete) → next: s14 (pending) | coverage: partial**
  ```
- 保持 `index.md` 为轻量级入口点，而非巨型详尽注册表。
- 默认选择满足工作流的最小连贯变更集。
- 如果请求可能匹配多个工作流，选择保留用户意图的最窄工作流。
- 如果工作流写入文件，始终报告按创建、更新或未变化分组的触及路径。
- 如果查询结果值得保存，在保存回工作空间之前先询问。
- **临时文件必须写入 `raw/.tmp/`。** 所有工作流产生的临时脚本、
  中间 JSON 数据、缓存文件等，一律放入项目内的 `raw/.tmp/` 目录。
  禁止写入 `/tmp/`（Windows 不可靠）、项目根目录、或其他系统临时目录。
  `raw/.tmp/` 内容为会话级别，不保证持久化，不计入 wiki 知识。
- 写入 wiki 的每条事实声明都必须包含来源锚点。
  格式：`→ [来源: raw/sources/filename.md]` 或 `→ [来源: filename.md:第XX页]`。
  示例：`→ [来源: RA4M2_manual.pdf, 第134页]`。任何声明都不能
  没有可追溯的来源。这适用于所有 wiki 页面和所有生成 wiki 内容的工作流。
- 每次查询回答都必须声明其来源。先阅读 wiki 再回答。
  回答后，说明哪些 wiki 页面为回答提供了信息，即使回答也利用了训练知识。
  如果 wiki 对某个主题没有记录，应如实说明，而非编造一个 wiki 支持的答案。
</global_rules>

## 安全规则

- **不修改 raw 层** — capture 后 raw/ 内容不可更改，源文件永远是 traceable 锚点
- **不静默丢弃信息** — 矛盾和不一致必须显式 surface，不悄悄选择一个版本
- **不经用户确认不删除** — curate/review 发现孤儿页面或重复页面时，必须先报告再处理
- **不暴露敏感内容** — source anchor 中不包含内网路径、密钥或凭证
- **人类修正优先** — 用户说"这个页面错了"等同于最高优先级信号，立即响应

<large_file_protocol>
当一次性从头到尾读取某个来源不切实际或浪费时，将其视为大型来源。

常见场景：
- 长篇手册和参考 PDF
- 可能需要 OCR 的扫描 PDF
- 大型表格、电子表格或寄存器映射表
- 日志、跟踪记录或长篇幅生成报告
- 压缩包或多附件来源包

规则：
- 首先捕获原始文件并保持其在 `raw/` 下不可变

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meng-jinglei/llm-wiki](https://github.com/meng-jinglei/llm-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
