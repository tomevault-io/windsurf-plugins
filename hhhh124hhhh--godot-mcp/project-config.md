---
trigger: always_on
description: 本文件为 Claude Code (claude.ai/code) 在此代码库中工作时提供指导。
---

# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 在此代码库中工作时提供指导。

## 架构概览

这是一个 **Godot 引擎 + Claude AI 集成**项目，使用模型上下文协议 (MCP)。项目采用 **客户端-服务器架构**：

- **Godot 插件 (客户端)**: 位于 `addons/godot_mcp/` - 作为 EditorPlugin 在 Godot 中运行
- **MCP 服务器 (后端)**: 位于 `server/` - 使用 FastMCP 的 Node.js/TypeScript 服务器

### 通信流程
```
Claude ↔ MCP 服务器 (FastMCP) ↔ WebSocket ↔ Godot 插件
```

Godot 插件创建 WebSocket 服务器（默认端口 9080），接受来自 MCP 服务器的连接。所有命令都通过此 WebSocket 连接使用 JSON 消息格式传输。

## 构建和运行命令

### MCP 服务器开发
- **安装依赖**: `cd server && npm install`
- **服务器构建**: `cd server && npm run build` (将 TypeScript 编译到 `dist/`)
- **启动服务器**: `cd server && npm run start`
- **开发模式**: `cd server && npm run dev` (使用 nodemon 自动重建)
- **类型检查**: `cd server && npx tsc --noEmit` (仅类型检查，不输出文件)

### Godot 项目
- **打开项目**: 在 Godot Editor 中打开 `project.godot`
- **插件位置**: `addons/godot_mcp/`
- **启用插件**: 项目设置 → 插件 → 启用 "Godot MCP"

### 测试连接
1. 启动 MCP 服务器: `cd server && npm run dev`
2. 在 Godot Editor 中打开项目（插件自动启动）
3. 检查 MCP 服务器日志，确认 WebSocket 连接到 `ws://localhost:9080`

## 核心组件

### Godot 插件端 (`addons/godot_mcp/`)
- **`mcp_server.gd`**: 主 EditorPlugin，创建 WebSocket 服务器
- **`command_handler.gd`**: 将 JSON 命令路由到相应的处理器
- **`commands/`**: 命令处理器（node_commands.gd, script_commands.gd, scene_commands.gd 等）
- **`ui/mcp_panel.gd`**: 编辑器 UI 面板，显示连接状态和日志

### MCP 服务器端 (`server/src/`)
- **`index.ts`**: FastMCP 服务器入口点，注册所有工具/资源
- **`utils/godot_connection.ts`**: WebSocket 连接管理，支持自动重连
- **`tools/`**: MCP 工具定义（node_tools.ts, script_tools.ts 等）
- **`resources/`**: MCP 资源提供器（scene_resources.ts, script_resources.ts 等）

## 开发模式

### 添加新命令
1. **Godot 端**: 在 `addons/godot_mcp/commands/` 中创建新命令类
   - 继承基础命令模式
   - 在 `command_handler.gd` 中注册
   - 遵循 snake_case 命名规范

2. **MCP 服务器端**: 在 `server/src/tools/` 中创建工具
   - 使用 FastMCP 工具构建器模式
   - 在 `server/src/index.ts` 中注册
   - 使用 Zod 进行参数验证

### 消息协议
**命令格式** (MCP → Godot):
```json
{
  "type": "command_name",
  "params": { /* 命令参数 */ },
  "commandId": "unique_id"
}
```

**响应格式** (Godot → MCP):
```json
{
  "status": "success|error",
  "result": { /* 响应数据 */ },
  "commandId": "matching_id"
}
```

## 代码风格指南

### TypeScript (服务器)
- 变量、方法和函数名使用 camelCase
- 类/接口使用 PascalCase
- 强类型：避免 `any` 类型，使用 Zod 进行运行时验证
- 优先使用 async/await 而非 Promise 链
- 导入结构：Node 模块在前，然后是本地模块
- 使用 ES 模块 (package.json 中设置 "type": "module")

### GDScript (Godot)
- 变量、方法和函数名使用 snake_case
- 类名使用 PascalCase
- 尽可能使用类型提示：`var player: Player`
- 遵循 Godot 单例约定（如 `Engine`, `OS`）
- 节点间通信优先使用信号
- 错误处理：开发检查使用断言和 `assert()`

## GDScript 缩进规范

### 基本缩进标准

#### 官方推荐标准
- **缩进字符**: 必须使用 **制表符 (Tab)**，而非空格
- **缩进宽度**: 显示为4个字符宽度（编辑器配置）
- **缩进层级**: 每个嵌套层级使用一个Tab
- **对齐**: 严格使用Tab缩进，避免Tab和空格混用

**为什么使用Tab而非空格？**
- Godot官方编辑器默认支持和推荐Tab缩进
- 文件大小更小，解析性能更好
- 避免空格/制表符混用导致的显示问题
- 团队协作时缩进一致性更容易保证

#### 具体缩进规则

**1. 类级别声明**
```gdscript
extends Node2D
class_name MyClass

# 这些声明不应该有前导缩进
var variable: int = 0
const CONSTANT: float = 1.0
signal my_signal()

func _ready() -> void:
    # 函数定义不应该有前导缩进
    pass
```

**2. 函数定义**
```gdscript
func my_function(param: int) -> void:
    # 函数体内的代码使用一个Tab缩进
    if condition:
        # 嵌套代码块使用两个Tab缩进
        nested_call()

    # 同级别的代码保持一个Tab缩进
    other_call()

static func static_function() -> void:
    # 静态函数同样遵循缩进规则
    pass
```

**3. 特殊情况处理**

**长行对齐**:
```gdscript
func long_function_call(
    param1: String,
    param2: int,
    param3: bool
) -> void:
    # 函数参数换行时，对齐到函数名位置
    function_body()
```

**链式调用**:
```gdscript
var tween = create_tween()
tween.tween_property(
    object, "property", value, duration
).set_trans(Tween.TRANS_SINE).set_ease(Tween.EASE_IN_OUT)
```

**数组/字典**:
```gdscript
var array = [
    item1,
    item2,
    item3
]

var dictionary = {
    "key1": value1,
    "key2": value2,
    "key3": value3
}
```

### IDE 配置指南

#### VS Code
在项目 `.vscode/settings.json` 中添加：
```json
{
    "editor.insertSpaces": false,
    "editor.tabSize": 4,
    "editor.detectIndentation": false,
    "[gdscript]": {
        "editor.insertSpaces": false,
        "editor.tabSize": 4,
        "editor.renderWhitespace": "boundary"
    }
}
```

#### Godot 编辑器
- **自动缩进**: 启用 (默认)
- **缩进大小**: 4 空格宽度
- **使用制表符**: 启用 (默认)
- **显示空白字符**: 建议启用以发现缩进问题

#### 其他编辑器配置

**Sublime Text**:
```json
{
    "translate_tabs_to_spaces": false,
    "tab_size": 4,
    "detect_indentation": false,
    "show_whitespace": true
}
```

**Vim/Neovim**:
```vim
set noexpandtab
set tabstop=4
set shiftwidth=4
set list
set listchars=tab:»·,trail:·
```

### 团队协作规则

#### 代码审查清单
在代码审查时，必须检查以下缩进问题：
- [ ] 文件中是否混用空格和制表符
- [ ] 函数定义前是否有多余的缩进
- [ ] 嵌套层级缩进是否正确
- [ ] 长行对齐是否一致
- [ ] 注释缩进是否与代码对齐

#### 协作流程规范

**新文件创建**:
1. 确保编辑器已正确配置缩进设置
2. 新文件开始时检查缩进是否使用Tab
3. 编写过程中定期显示空白字符检查

**文件修改**:
1. 打开文件时检查现有缩进模式
2. 修改时保持与现有代码一致的缩进
3. 提交前运行缩进检查工具

**Git 配置**:
```gitconfig
# .gitattributes 文件配置
*.gd text eol=lf
*.gdscript text eol=lf
```

### 常见错误预防

#### 错误识别指南

**1. 制表符和空格混用**
```gdscript
# ❌ 错误：混用空格和制表符
func bad_example() -> void:
	var variable = 0    # 这里可能使用了空格
        another_var = 1   # 这里可能使用了不同的缩进

# ✅ 正确：统一使用制表符
func good_example() -> void:
	var variable = 0
	another_var = 1
```

**2. 函数定义缩进错误**
```gdscript
# ❌ 错误：函数定义前有缩进
	func bad_function() -> void:
		pass

# ✅ 正确：函数定义无前导缩进
func good_function() -> void:
	pass
```

**3. 嵌套缩进不一致**
```gdscript
# ❌ 错误：缩进层级混乱
func bad_function() -> void:
if condition:
do_something()
		nested_call()

# ✅ 正确：清晰的层级缩进

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hhhh124hhhh/godot-mcp](https://github.com/hhhh124hhhh/godot-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
