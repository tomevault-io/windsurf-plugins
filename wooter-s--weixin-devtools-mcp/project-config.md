---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

微信开发者工具自动化 MCP 服务器，提供31个工具用于微信小程序的自动化测试。基于 TypeScript 和 `miniprogram-automator` SDK 实现。

## Common Commands

### 开发和构建
```bash
# 构建项目（TypeScript → JavaScript + 设置可执行权限）
npm run build

# 开发模式（监听文件变化自动重新构建）
npm run watch

# 使用 MCP Inspector 调试
npm run inspector
```

### 测试

项目采用分层测试架构，参考 chrome-devtools-mcp 模式：

```
tests/
├── protocol/          # 协议层测试（需要MCP服务器）
│   ├── server.test.ts
│   └── index.test.ts
├── tools/            # 工具逻辑测试（直接调用handler，无需服务器）
│   ├── connection.test.ts
│   ├── console.test.ts
│   ├── navigate.test.ts
│   ├── network.test.ts
│   ├── page.test.ts
│   └── screenshot.test.ts
├── integration/      # 集成测试（需要真实环境）
│   ├── connect-devtools.integration.test.ts
│   ├── console.integration.test.ts
│   ├── enhanced-connection.integration.test.ts
│   ├── navigation.integration.test.ts
│   ├── network.integration.test.ts
│   └── network-auto-start.integration.test.ts
└── utils/            # 测试工具
    └── test-utils.ts
```

**测试命令**：

```bash
# 单元测试（协议 + 工具 + 工具类，224个测试）
npm test

# 分类运行单元测试
npm run test:protocol      # 协议层测试（19个）
npm run test:tools         # 工具逻辑测试（196个）

# 集成测试（需要微信开发者工具 + playground/wx/）
npm run test:integration   # 46个集成测试

# 所有测试（单元 + 集成）
npm run test:all

# 测试覆盖率
npm run test:coverage

# 监听模式
npm run test:watch                # 单元测试监听
npm run test:integration:watch    # 集成测试监听
```

**集成测试要求**：
- 微信开发者工具已安装并开启自动化功能
- 测试项目位于 `playground/wx/`
- 通过环境变量 `RUN_INTEGRATION_TESTS=true` 控制执行
- 不设置该环境变量时，集成测试会自动跳过

### 运行单个测试

```bash
# 协议测试
npx vitest tests/protocol/server.test.ts

# 工具测试
npx vitest tests/tools/console.test.ts

# 集成测试
RUN_INTEGRATION_TESTS=true npx vitest tests/integration/console.integration.test.ts

# 指定测试用例
npm test -- tests/tools/console.test.ts -t "测试用例名称"
```

### 诊断与手工验证脚本

```bash
# 诊断类脚本（scripts/diagnostics）
npm run diagnose:devtools-connection
npm run diagnose:mcp-config
npm run diagnose:connection-flow

# 手工验证脚本（tests/manual）
npm run test:manual:mpx-runtime
npm run test:manual:network-interception
npm run test:manual:screenshot
npm run test:manual:screenshot-diagnostic
```

## Architecture

### MCP 服务器入口点

**`build/server.js`**
- 源文件：`src/server.ts`
- 特点：完全模块化的工具系统，代码简洁
- 代码量：~245行
- 工具处理：所有31个工具统一通过 `allTools` 数组和 `ToolDefinition` 框架处理
- 配置：`npm install -g weixin-devtools-mcp` 默认使用此入口（package.json bin配置）

### 模块化工具系统

核心设计模式参考 chrome-devtools-mcp：

```
src/tools/
├── ToolDefinition.ts    # 核心框架
│   ├── defineTool()     # 工具定义辅助函数
│   ├── ToolContext      # 共享状态接口（5个字段）
│   ├── ToolHandler      # 工具处理器类型
│   └── ToolResponse     # 响应构建接口
│
├── index.ts             # 统一导出 allTools[] (31个工具)
│
└── [8个功能模块]
    ├── connection.ts    # 连接管理（3工具）
    ├── page.ts          # 页面查询（2工具：query_selector、wait_for）
    ├── snapshot.ts      # 页面快照（1工具）
    ├── input.ts         # 交互操作（7工具）
    ├── assert.ts        # 断言验证（5工具）
    ├── navigate.ts      # 页面导航（6工具）
    ├── console.ts       # Console监听（6工具：含两阶段查询）
    ├── network.ts       # 网络监控（5工具）
    ├── screenshot.ts    # 截图工具（1工具）
    └── diagnose.ts      # 诊断工具（4工具）
```

**工具定义模式**：
```typescript
// 每个工具都遵循相同的定义模式
export const exampleTool = defineTool({
  name: "tool_name",
  description: "工具描述",
  schema: z.object({ /* Zod schema */ }),
  handler: async (request, response, context) => {
    // 1. 从 context 获取共享状态
    // 2. 执行业务逻辑
    // 3. 通过 response.appendResponseLine() 返回结果
    // 4. 更新 context 状态（自动同步到全局）
  }
});
```

### 状态管理（ToolContext）

所有工具通过 `ToolContext` 共享5个关键状态：

1. **`miniProgram`**: MiniProgram实例（来自miniprogram-automator）
2. **`currentPage`**: 当前活动页面实例
3. **`elementMap`**: Map<uid, ElementMapInfo> - 元素UID到选择器的映射
4. **`consoleStorage`**: Console消息和异常存储（监听状态 + 消息数组）
5. **`networkStorage`**: 网络请求拦截数据（监听状态 + 请求数组 + 原始方法）

**关键设计**：
- 工具间通过 context 传递状态，无全局变量污染
- `elementMap` 支持 UID引用机制（`get_page_snapshot`生成UID，`click`等工具使用UID操作元素）
- 网络监控在 `connect_devtools_enhanced` 连接时自动启动

### UID 引用机制

支持跨工具的元素引用：

```typescript
// 1. 获取页面快照（生成所有元素的UID）
get_page_snapshot()
// 输出：{ uid: "button.submit", tagName: "button", ... }

// 2. 使用UID操作元素
click({ uid: "button.submit" })
input_text({ uid: "input#username", text: "user" })
assert_text({ uid: ".message", text: "成功" })
```

UID生成规则：优先使用 id > class > nth-child 构建稳定的CSS选择器路径。

## Technical Details

### 关键依赖
- `@modelcontextprotocol/sdk` (v0.6.0) - MCP协议实现
- `miniprogram-automator` (^0.12.1) - 微信小程序自动化SDK
- `zod` + `zod-to-json-schema` - 参数验证和schema转换
- `vitest` - 测试框架

### TypeScript配置
- Target: ES2022, Module: Node16 (ESM)
- `"type": "module"` in package.json
- 严格模式启用
- 输出目录：`./build`

### 构建过程
1. TypeScript编译（`tsc`）
2. 自动设置可执行权限（`build/server.js`）
3. prepare hook确保发布前构建

### MCP服务器配置

**macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
**Windows**: `%APPDATA%/Claude/claude_desktop_config.json`

```json
{
  "mcpServers": {
    "weixin-devtools-mcp": {
      "command": "npx",
      "args": ["-y", "weixin-devtools-mcp"]
    }
  }
}
```

或使用本地路径（开发者）：
```json
{
  "mcpServers": {
    "weixin-devtools-mcp": {
      "command": "/path/to/weixin-devtools-mcp/build/server.js"
    }
  }
}
```

## Development Notes

### 添加新工具

1. 在 `src/tools/` 下选择合适的功能模块（或创建新模块）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wooter-s/weixin-devtools-mcp](https://github.com/wooter-s/weixin-devtools-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-21 -->
