---
trigger: always_on
description: 此文件为在此代码库中工作的 Claude Code (claude.ai/code) 提供指导。
---

# CLAUDE.md

此文件为在此代码库中工作的 Claude Code (claude.ai/code) 提供指导。

## 项目概述

这是一个基于 Wails v2 + Vue 3 + TypeScript 构建的桌面开发者工具应用。它提供了一系列常用的开发者实用工具，如 JSON/XML 编辑器、时间转换器、URL 工具、Base64 转换、二维码生成、颜色转换等。

## 架构

**后端 (Go):**
- `main.go` - 入口点，包含 Wails 应用配置
- `app.go` - 主应用结构体和基本方法
- `json_processor.go` - JSON 处理工具
- `xml_processor.go` - XML 处理工具
- 使用 Wails v2 作为桌面应用框架

**前端 (Vue 3):**
- `frontend/src/App.vue` - 主 Vue 应用组件
- `frontend/src/views/` - 各个工具视图 (JsonEditor, TimeConverter, ColorConverter 等)
- `frontend/src/components/` - 可复用的 Vue 组件
- `frontend/src/stores/` - Pinia 状态管理
- `frontend/src/router/` - Vue Router 配置
- 使用 Element Plus 作为 UI 组件库
- 使用 Monaco Editor 进行代码编辑
- 使用 Vite 作为构建工具

## 常用命令

### 开发
```bash
# 启动开发服务器（热重载）
wails dev

# 安装前端依赖
cd frontend && npm install

# 安装 Go 依赖
go mod tidy
```

### 构建
```bash
# 为当前平台构建
wails build

# 为 macOS 通用版构建
make build-mac
# 或
wails build -platform=darwin/universal

# 为 Windows 构建
make build-win
# 或
wails build -platform=windows/amd64

# 带调试/开发工具构建
make build-mac-dev
make build-win-dev
```

### 前端开发
```bash
cd frontend

# 开发服务器
npm run dev

# 仅构建前端
npm run build

# 类型检查
vue-tsc --noEmit
```

### macOS 分发
```bash
# 创建 DMG 包
make dmg

# 清理构建产物
make clean
```

## 项目结构

- 后端 Go 文件位于根目录
- 前端 Vue 应用位于 `frontend/` 目录
- Wails 配置文件 `wails.json`
- 构建输出至 `build/bin/`
- 文档和图片位于 `docs/`

## 关键依赖

**Go:**
- `github.com/wailsapp/wails/v2` - 桌面应用框架
- `github.com/gopherjs/gopherjs` - JS 互操作工具

**前端:**
- Vue 3 + TypeScript
- Element Plus (UI 组件)
- Monaco Editor (代码编辑)
- Pinia (状态管理)
- 各种特定工具的实用库

## 开发注意事项

- 应用使用嵌入式资源 (`//go:embed all:frontend/dist`)
- 前端通过 Wails 绑定与 Go 后端通信
- 多个工具视图组织为独立的 Vue 组件
- 工具状态和偏好设置通过 Pinia 存储管理
- 应用支持拖拽重排序工具卡片
- 所有工具都支持实时处理和预览

## UI 设计规范

### 统一的模块布局风格

所有工具模块都应该采用统一的布局风格，参考 `Base64Text.vue` 作为标准模板：

**顶部导航栏结构：**
```vue
<div class="top-header">
  <div class="tab-nav">
    <!-- 左侧：功能按钮/标签页 -->
    <button class="tab-btn" :class="{ active: condition }">标签</button>
    <button class="action-btn">示例</button>
    <!-- 其他控件如下拉框 -->
  </div>
  <div class="tab-actions">
    <!-- 右侧：清空按钮 -->
    <button class="clear-btn" title="清空">× 清空</button>
  </div>
</div>
```

**内容区域结构：**
```vue
<div class="content-layout">
  <div class="input-panel">
    <textarea class="text-area" />
  </div>
  <div class="output-panel">
    <textarea class="text-area" />
  </div>
</div>
```

### 样式规范

**顶部导航栏样式（重要）：**
```css
.top-header {
  display: flex;
  justify-content: space-between;
  align-items: stretch;
  padding: 0;
  margin: 0;
  background: #f8f9fa;  /* 统一背景色 */
  height: 28px;         /* 统一高度 */
  flex-shrink: 0;
}

.tab-nav {
  display: flex;
  align-items: stretch;
  background: #f8f9fa;   /* 统一背景色 */
}

.tab-actions {
  display: flex;
  align-items: stretch;  /* 改为 stretch，不是 center */
  background: #f8f9fa;   /* 统一背景色，不是白色 */
}
```

**按钮样式（必须严格遵循）：**
- 统一高度：`height: 100%` （继承父容器28px）
- 字体大小：`font-size: 10px`
- 颜色：`color: #6c757d`
- 背景：`background: #f8f9fa`
- 圆角：`border-radius: 0` （无圆角）
- 内边距：`padding: 0 10px`
- 最小宽度：`min-width: 45px`
- hover 状态：`background: #e9ecef`
- active 状态：`background: #ffffff; color: #212529`
- **边框规则**：
  - 第一个按钮：`border-left: 1px solid #d1d5db`
  - 相邻按钮：`border-left: 1px solid #d1d5db`（避免重复）
  - 最后一个按钮：`border-right: 1px solid #d1d5db`
  - 清空按钮：`border-left: 1px solid #d1d5db; border-right: 1px solid #d1d5db; border-top: none; border-bottom: none`

**按钮边框完整示例：**
```css
/* 标签页按钮组 */
.tab-btn:first-child {
  border-left: 1px solid #d1d5db;
}
.tab-btn + .tab-btn {
  border-left: 1px solid #d1d5db;
}
.tab-btn:last-child {
  border-right: 1px solid #d1d5db;
}

/* 示例按钮 */
.action-btn {
  border-left: 1px solid #d1d5db;
  border-right: 1px solid #d1d5db;
}

/* 清空按钮 */
.clear-btn {
  border-top: none;
  border-bottom: none;
  border-left: 1px solid #d1d5db;
  border-right: 1px solid #d1d5db;
}
```

**文本区域样式：**
- 字体：`font-family: 'SF Mono', Monaco, 'Cascadia Code', 'Roboto Mono', Consolas, 'Courier New', monospace`
- 字体大小：`font-size: 11px`
- 行高：`line-height: 1.4`
- 颜色：`color: #212529`
- 内边距：`padding: 12px`
- 占位符颜色：`color: #9ca3af`

**布局样式：**
- 容器内边距：`padding: 16px`
- 顶部导航栏：`height: 28px; margin-bottom: 16px`
- 左右面板间距：`gap: 16px`
- 边框：`border: 1px solid #d1d5db`
- 背景：`background: #ffffff`

**响应式断点：**
- 桌面端：默认左右布局
- 移动端：`@media (max-width: 768px)` 改为上下布局

### 功能规范

**自动处理：**
- 输入变化时自动处理/转换，不需要手动点击转换按钮
- 提供示例按钮，点击加载预设示例数据
- 清空按钮清除所有输入输出内容
- 从其他页面切换回来时，如果有内容应自动处理

**Monaco Editor 配置：**
```javascript
const editorOptions = {
  fontSize: 11,
  tabSize: 2,
  minimap: { enabled: false },
  scrollBeyondLastLine: true,
  automaticLayout: true,
  wordWrap: 'on',
  lineNumbers: 'on',
  readOnly: true // 输出区域设为只读
}
```

### 重要提醒

- **新增工具模块时必须遵循以上设计规范**
- **所有按钮、文本框、布局都要与现有模块保持一致**
- **特别注意按钮边框规则，确保按钮之间有正确的分隔线**
- **顶部导航栏背景必须统一为 `#f8f9fa`**
- **避免添加不必要的 UI 元素，保持界面简洁**
- **优先考虑自动化处理，减少用户手动操作**

### 开发新模块时的检查清单

创建新工具模块时，请确保：

1. ✅ 顶部导航栏背景色为 `#f8f9fa`
2. ✅ 所有按钮高度为 `100%`，字体大小为 `10px`
3. ✅ 按钮边框按规则设置，确保有正确的分隔线
4. ✅ 清空按钮样式正确（左右边框，上下无边框）
5. ✅ hover效果统一为 `#e9ecef`
6. ✅ 容器使用 `margin: 0 !important; padding: 0 !important`
7. ✅ 文本区域使用等宽字体和统一的字体大小
8. ✅ 支持自动处理和示例数据加载

## 文件保存功能

### 后端实现 (app.go)

使用通用的 `SaveFile` 方法，支持保存任何类型的文件：

```go
// 文件过滤器类型
type FileFilter struct {
	DisplayName string `json:"displayName"`
	Pattern     string `json:"pattern"`
}

// 保存文件选项
type SaveFileOptions struct {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zzdylan/dev-tools](https://github.com/zzdylan/dev-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
