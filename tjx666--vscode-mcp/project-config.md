---
trigger: always_on
description: VSCode MCP 工具开发指南，包含开发流程、架构、步骤、质量标准和最佳实践
---


# VSCode MCP 工具开发指南

本指南总结了在 VSCode MCP Bridge 项目中开发和维护工具的完整流程，包括从基础实现到质量优化的全套最佳实践。

## 开发流程

### 核心原则

**开发顺序**：

```plaintext
接口定义 (IPC) → 实现层 (Extension) → 工具层 (MCP Server) → 质量优化
```

**为什么要按这个顺序？**

- IPC 层定义了类型契约，必须最先完成
- Extension 层依赖 IPC 的类型定义
- MCP Server 层调用 Extension 的服务
- 质量优化确保符合 MCP 官方标准

**重要提醒**: 每个开发阶段完成后，都必须进行 **编译和构建验证**，确保代码质量和依赖关系正确。

## 工具架构

### 三层架构

```plaintext
MCP Client ↔ MCP Server ↔ IPC Layer ↔ VSCode Extension ↔ VSCode API
```

**职责分工**：

- **IPC Layer**: 定义类型契约和通信协议
- **Extension Layer**: 实现具体的 VSCode 操作逻辑
- **MCP Server Layer**: 提供标准化的 MCP 工具接口

## 开发步骤

### 1. IPC 层：定义接口

#### 1.1 创建事件定义文件

在 [packages/vscode-mcp-ipc/src/events/](mdc:packages/vscode-mcp-ipc/src/events/) 目录中创建事件定义文件：

```typescript
// packages/vscode-mcp-ipc/src/events/your-tool.ts
import { z } from 'zod';

export const YourToolInputSchema = z
  .object({
    param1: z.string().describe('参数描述'),
    param2: z.boolean().optional().default(true).describe('可选参数'),
  })
  .strict();

export const YourToolOutputSchema = z
  .object({
    result: z.string().describe('结果描述'),
  })
  .strict();

export type YourToolPayload = z.infer<typeof YourToolInputSchema>;
export type YourToolResult = z.infer<typeof YourToolOutputSchema>;
```

#### 1.2 注册到事件映射

更新 [packages/vscode-mcp-ipc/src/events/index.ts](mdc:packages/vscode-mcp-ipc/src/events/index.ts)：

```typescript
import type { YourToolPayload, YourToolResult } from './your-tool.js';

export * from './your-tool.js';

export interface EventMap {
  yourTool: {
    params: YourToolPayload;
    result: YourToolResult;
  };
}
```

#### 1.3 构建 IPC 包

```bash
cd packages/vscode-mcp-ipc && npm run build
```

### 2. Extension 层：实现服务

#### 2.1 创建服务实现

在 [packages/vscode-mcp-bridge/src/services/](mdc:packages/vscode-mcp-bridge/src/services/) 目录中实现服务：

```typescript
// packages/vscode-mcp-bridge/src/services/your-tool.ts
import type { EventParams, EventResult } from '@vscode-mcp/vscode-mcp-ipc';

export const yourTool = async (
  payload: EventParams<'yourTool'>,
): Promise<EventResult<'yourTool'>> => {
  try {
    const result = await someVSCodeOperation(payload.param1);
    return { result };
  } catch (error) {
    throw new Error(`操作失败: ${error}`);
  }
};
```

#### 2.2 注册服务

更新 [packages/vscode-mcp-bridge/src/extension.ts](mdc:packages/vscode-mcp-bridge/src/extension.ts)：

```typescript
import { YourToolInputSchema, YourToolOutputSchema } from '@vscode-mcp/vscode-mcp-ipc';
import { yourTool } from './services';

socketServer.register('yourTool', {
  handler: yourTool,
  payloadSchema: YourToolInputSchema,
  resultSchema: YourToolOutputSchema,
});
```

### 3. MCP Server 层：创建工具

#### 3.1 创建工具文件

在 [packages/vscode-mcp-server/src/tools/](mdc:packages/vscode-mcp-server/src/tools/) 目录中实现工具：

```typescript
// packages/vscode-mcp-server/src/tools/your-tool.ts
import type { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js';
import { createDispatcher, YourToolInputSchema } from '@vscode-mcp/vscode-mcp-ipc';
import { z } from 'zod';
import { formatToolCallError } from './utils.js';

const inputSchema = {
  workspace_path: z.string().describe('VSCode workspace path to target'),
  ...YourToolInputSchema.shape,
};

export function registerYourTool(server: McpServer) {
  server.registerTool(
    'your_tool',
    {
      title: 'Your Tool Title',
      description: 'Detailed description with usage scenarios and examples',
      inputSchema,
      annotations: {
        title: 'Your Tool Title',
        readOnlyHint: true,
        destructiveHint: false,
        idempotentHint: true,
        openWorldHint: false,
      },
    },
    async ({ workspace_path, param1, param2 }) => {
      try {
        const dispatcher = createDispatcher(workspace_path);
        const result = await dispatcher.dispatch('yourTool', { param1, param2 });

        return {
          content: [
            {
              type: 'text' as const,
              text: `✅ 操作成功: ${result.result}`,
            },
          ],
        };
      } catch (error) {
        return formatToolCallError('Your Tool Title', error);
      }
    },
  );
}
```

## 质量标准

### 错误处理标准

遵循 MCP 官方错误处理规范，使用统一的错误处理函数：

```typescript
// packages/vscode-mcp-server/src/tools/utils.ts
export function formatToolCallError(toolName: string, error: unknown) {
  return {
    isError: true, // MCP 官方要求
    content: [
      {
        type: 'text' as const,
        text: `❌ ${toolName} failed: ${String(error)}`,
      },
    ],
  };
}
```

### Tool Annotations 标准

根据 MCP 官方规范配置：

| Annotation        | 类型    | 默认值 | 描述               |
| ----------------- | ------- | ------ | ------------------ |
| `title`           | string  | -      | 人性化标题         |
| `readOnlyHint`    | boolean | false  | 是否只读操作       |
| `destructiveHint` | boolean | true   | 是否可能破坏性操作 |
| `idempotentHint`  | boolean | false  | 是否幂等操作       |
| `openWorldHint`   | boolean | true   | 是否与外部世界交互 |

## 最佳实践

### 1. 命名规范

- **事件名**: camelCase (`getDiagnostics`, `openFiles`)
- **工具名**: snake_case (`get_diagnostics`, `open_files`)
- **文件名**: kebab-case (`get-diagnostics.ts`, `open-files.ts`)

### 2. 错误处理

- 统一使用 `formatToolCallError` 函数
- 必须设置 `isError: true`
- 提供有意义的错误信息

### 3. Schema 设计

- 使用 `.describe()` 为所有参数添加描述
- 设置合理的默认值
- 使用 `.strict()` 确保类型安全
- MCP 工具层必须复用 IPC 层的 Schema

### 4. 构建验证


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tjx666/vscode-mcp](https://github.com/tjx666/vscode-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
