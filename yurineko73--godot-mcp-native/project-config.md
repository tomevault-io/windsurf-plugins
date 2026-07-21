---
trigger: always_on
description: 一个 **Godot 4.6 EditorPlugin**（位于 `addons/godot_mcp/`），在 Godot 内部原生实现了 MCP（Model Context Protocol）服务器，无需 Node.js 依赖。提供 **154 个工具**（30 核心 + 124 补充），分为 6 大类，供 AI 助手读取和修改项目。
---

# AGENTS.md — Godot MCP 项目指南

## 项目简介
一个 **Godot 4.6 EditorPlugin**（位于 `addons/godot_mcp/`），在 Godot 内部原生实现了 MCP（Model Context Protocol）服务器，无需 Node.js 依赖。提供 **154 个工具**（30 核心 + 124 补充），分为 6 大类，供 AI 助手读取和修改项目。

- **插件入口**：`addons/godot_mcp/mcp_server_native.gd`（继承 `EditorPlugin`）
- **作者**：yurineko73 | **版本**：1.0.7-pre1
- **许可证**：MIT
- **渲染器**：GL Compatibility

## PowerShell 7.x 路径
系统 PowerShell 版本可能变化，每次会话请验证确切的版本号：
```
& "C:\Program Files\WindowsApps\Microsoft.PowerShell_*_x64__8wekyb3d8bbwe\pwsh.exe"
```
用 `(Get-Command pwsh).Source` 解析路径。如果 WindowsApps 路径不可用，回退到系统默认 PowerShell。

## 代码语言政策
- **任何代码文件中不得出现中文字符** — 注释、字符串、标识符全部使用英文。
- 中文仅允许出现在：`AGENTS.md`、`README.zh.md` 和 `docs/`（翻译/计划文档）。

## 命令

### 运行 Godot 项目
在 Godot 编辑器（4.6.x，GL Compatibility 渲染器）中打开 `project.godot`。

### GUT 单元测试
```powershell
& "F:\Godot\Godot_v4.6.1-stable_win64.exe" --headless --path "." -s addons/gut/gut_cmdln.gd -gdir=res://test/unit/ -ginclude_subdirs -gexit
```
配置：`.gutconfig.json` — dirs: `res://test/unit/`, `log_level: 2`, `should_exit_on_finish: false`。

### 集成测试（Python）
```powershell
cd test/integration
python test_runtime_probe_flow.py
```
Python 测试会启动 Godot 4.6.2（`C:\SourceCode\Godot_v4.6.2-stable_mono_win64\...`），通过 HTTP MCP（端口 9080）进行通信。

## 架构

```
addons/godot_mcp/
├── mcp_server_native.gd       # EditorPlugin 入口 — 注册插件、管理生命周期
├── native_mcp/                 # 核心服务器引擎
│   ├── mcp_server_core.gd      # 中枢：工具注册、JSON-RPC 分发、信号总线
│   ├── mcp_transport_base.gd   # 传输层抽象基类
│   ├── mcp_http_server.gd      # HTTP/SSE 传输（默认端口 9080）
│   ├── mcp_stdio_server.gd     # stdio 传输（供 Claude Desktop 等使用）
│   ├── mcp_types.gd            # JSON-RPC 和 MCP 协议常量、MCPTool 数据类
│   ├── mcp_tool_classifier.gd  # 工具分类映射：{core/supplementary, group}（CORE_MAX_COUNT=30）
│   ├── mcp_resource_manager.gd # 资源读取/列表/订阅
│   ├── mcp_debugger_bridge.gd  # Godot 调试器 ↔ MCP 桥梁（断点、栈帧、变量）
│   ├── mcp_auth_manager.gd     # HTTP Bearer Token 认证
│   ├── config_manager.gd       # 插件配置读写
│   ├── settings_manager.gd     # 编辑器设置持久化
│   ├── tool_state_manager.gd   # 每个工具的状态管理
│   └── translation_manager.gd  # 国际化支持
├── runtime/
│   └── mcp_runtime_probe.gd    # Autoload 单例，用于运行时检查（动画、音频、着色器、瓦片地图、输入）
├── tools/                      # 工具实现（每个分类一个文件）
│   ├── node_tools_native.gd    # 20 个工具 — 创建/删除/更新/复制/移动/重命名节点、信号、分组、锚点预设、批量操作、场景审计
│   ├── script_tools_native.gd  # 14 个工具 — 读取/写入/创建/附加/分析/验证/执行脚本、符号索引、搜索
│   ├── scene_tools_native.gd   # 8 个工具 — 创建/保存/打开/关闭场景、结构查看、列表
│   ├── editor_tools_native.gd  # 16 个工具 — 运行/停止、状态、截图、信号、导出、选择、查看器
│   ├── debug_tools_native.gd   # 70 个工具 — 日志、断点、栈帧/变量、性能分析器、运行时探针、动画/音频/着色器/瓦片地图运行时控制
│   └── project_tools_native.gd # 26 个工具 — 项目信息/设置、资源、输入映射、自动加载、全局类、测试运行器、诊断
├── ui/
│   ├── mcp_panel_native.gd     # 主停靠面板（VBoxContainer）— 启动/停止、传输配置、日志查看、工具管理
│   ├── mcp_tool_item.gd        # 单个工具开关 UI
│   └── mcp_tool_group_item.gd  # 工具组折叠/展开 UI
└── utils/
    ├── node_utils.gd           # 节点查找辅助函数
    ├── path_validator.gd       # 路径验证
    ├── resource_utils.gd       # 资源 I/O 工具
    ├── script_utils.gd         # 脚本工具
    └── vibe_coding_policy.gd   # Vibe Coding 模式守卫（allow_ui_focus / allow_window）
```

## 规范

### GDScript 风格
- 变量、方法、函数名使用 `snake_case`
- 类名使用 `PascalCase`（`class_name ClassName`）
- 必须使用类型提示：`var player: Player`、`func greet() -> String:`
- 优先使用 Signal 进行节点间解耦通信
- 编辑器插件脚本使用 `@tool`
- 非节点工具类继承 `RefCounted`，插件入口继承 `EditorPlugin`
- GUT 测试文件使用 `extends "res://addons/gut/test.gd"`（不要用 `class_name` — GUT CLI 限制）

### 错误处理
- 工具处理函数返回错误字典：`{"error": "message"}`
- 生产代码中少用 `assert()`
- 每个工具处理函数顶部先验证参数

### 注释
- 中文注释仅允许在 AGENTS.md / README.zh.md / docs 中
- 所有 `.gd`、`.cs`、`.py` 文件：仅限英文

### 测试（强制）
- **每次代码变更必须包含测试更新**，位于 `test/` 目录：
  - **直接测试**：覆盖变动的逻辑（正常路径 + 边界情况 + 错误处理）
  - **影响范围测试**：识别并测试可能受影响的关联模块（签名变更、导出变量、信号等）
  - 禁止提交没有测试更新的 commit
- **单元测试**（`test/unit/`）：GUT 框架，工具测试文件位于 `test/unit/tools/`
- **集成测试**（`test/integration/`）：Python 脚本，通过 HTTP MCP 调用 Godot 4.6.2

### 新增工具流程

创建新 MCP 工具时，必须按顺序完成所有步骤。完整参考指南见 `docs/development/adding-new-tools.md`。

1. **实现处理器** — 在对应的 `*_tools_native.gd` 中创建 `_register_<name>()` 和 `_tool_<name>()` 函数，用 8 个参数调用 `server_core.register_tool()`（name, desc, input_schema, Callable, output_schema, annotations, category, group）
2. **注册到分类器** — 在 `mcp_tool_classifier.gd` 的 `_build_classifications()` 中添加条目，然后更新 `test_mcp_tool_classifier.gd` 中的工具总数和 supplementary 计数
3. **添加单元测试** — 在 `test/unit/tools/` 中覆盖缺失参数/无效参数/边界情况
4. **更新翻译文件** — 在 `translations/tool_descriptions.json` 和 `translations/tool_descriptions.csv` 中添加工具描述（中英文）
5. **更新文档** — `docs/current/tools-reference.md`（更新概览表格 + 添加工具条目）、`addons/godot_mcp/README.md` 和 `README.zh.md`
6. **验证** — 运行完整 GUT 测试套件，要求 0 失败

**注意：** supplementary 工具注册后默认禁用（`enabled = (category == "core")`），`tools/list` 不会返回它。用户需在 MCP 面板中手动启用，或在测试时用 `core.set_tool_enabled("tool_name", true)` 开启。

### 修改已有工具后的文档更新流程

每次修改已有工具（新增参数、返回值、行为变更）时，必须同步更新以下文档：

1. **`docs/current/tools-reference.md`**
   - 更新工具的**参数表**（新增/变更的参数名、类型、必需、描述）
   - 更新工具的**返回值表**（新增/变更的字段名、类型、描述）
   - 更新概览表中的分类计数（如调试工具 67→68）
   - 重编号后续工具（新增工具条目时）
2. **`README.md` 和 `README.zh.md`**（项目根目录和 `addons/godot_mcp/` 下）
   - 更新功能列表中的工具计数（`Comprehensive Tool Set` / `全面的工具集`）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yurineko73/Godot-MCP-Native](https://github.com/yurineko73/Godot-MCP-Native) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
