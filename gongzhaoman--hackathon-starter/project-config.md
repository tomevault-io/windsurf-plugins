---
trigger: always_on
description: validateSettings(): void {
---

# CLAUDE.md - 智能体平台开发指南

本文件为 Claude Code (claude.ai/code) 在此代码仓库中工作时提供开发流程指导。

## ⚡ 标准开发流程（重要）

**每次进行代码修改时必须严格按照以下顺序执行：**

### 1. 开发环境准备

```bash
# 启动开发环境
docker compose up --build -d

# 检查服务状态
docker compose ps
```

### 2. 代码修改流程

1. **修改代码** - 进行必要的功能开发或bug修复
2. **类型检查** - `pnpm --filter agent-api typecheck`
3. **运行测试** - `pnpm --filter agent-api test`（必须通过）
4. **代码检查** - `pnpm --filter agent-api lint`
5. **手动验证** - 确认功能正常工作

### 3. 文档更新流程

**完成代码开发后，必须按顺序更新以下文档：**

1. **更新英文README** - 如有新功能或命令变更
2. **更新中文README** - 保持与英文版本同步
3. **更新docs目录** - 更新相关技术文档（架构指南、测试指南等）
4. **更新CLAUDE.md** - 如有开发流程或命令变更

### 4. 提交前检查

```bash
# 最终检查（所有命令必须成功）
pnpm --filter agent-api typecheck
pnpm --filter agent-api test
pnpm --filter agent-api lint
```

## 📋 开发命令参考

### 根目录命令 (Turborepo)

- `pnpm dev` - 启动所有应用的开发模式
- `pnpm build` - 构建所有应用
- `pnpm lint` - 检查所有应用代码
- `pnpm format` - 使用Prettier格式化代码

### 应用特定命令

- `pnpm --filter agent-api <command>` - 在agent-api中运行命令
- `pnpm --filter agent-web <command>` - 在agent-web中运行命令

### Agent API (NestJS) 命令

- `pnpm --filter agent-api dev` - 启动API监听模式
- `pnpm --filter agent-api build` - 构建API
- `pnpm --filter agent-api test` - 运行单元测试
- `pnpm --filter agent-api test:e2e` - 运行端到端测试
- `pnpm --filter agent-api typecheck` - 类型检查
- `pnpm --filter agent-api lint` - 代码检查并自动修复

### 数据库命令

- `pnpm --filter agent-api db:generate` - 生成Prisma客户端
- `pnpm --filter agent-api db:migrate` - 运行数据库迁移
- `pnpm --filter agent-api db:push` - 推送模式变更
- `pnpm --filter agent-api db:studio` - 打开Prisma Studio
- `pnpm --filter agent-api db:seed` - 初始化数据库
- `pnpm --filter agent-api db:reset` - 重置并重新初始化数据库

### Agent Web (React + Vite) 命令

- `pnpm --filter agent-web dev` - 启动Web应用开发模式
- `pnpm --filter agent-web build` - 构建Web应用
- `pnpm --filter agent-web typecheck` - 类型检查
- `pnpm --filter agent-web lint` - ESLint代码检查

### Docker 开发命令

- `docker compose up --build -d` - 启动完整的Docker开发环境
- `docker compose ps` - 检查服务状态
- `docker compose down` - 停止所有服务
- `docker compose logs -f <service>` - 查看服务日志

## 🏗️ 系统架构概览

### Monorepo 架构

基于 Turborepo 的 Monorepo 架构，包含以下结构：

- **apps/agent-api**: 基于 Prisma ORM 的 NestJS 后端 API
- **apps/agent-web**: 基于 Vite 的 React 前端
- **packages/**: 共享包（UI组件、配置）

### Agent API (后端)

使用 NestJS 框架构建的模块化架构：

**核心模块：**

- **AgentModule** (`src/agent/`): 管理AI智能体及其配置
- **WorkflowModule** (`src/workflow/`): 处理基于DSL的工作流执行
- **ToolsModule** (`src/tool/`): 管理工具集和单个工具
- **KnowledgeBaseModule** (`src/knowledge-base/`): RAG向量数据库集成
- **LlamaIndexModule** (`src/llamaindex/`): LlamaIndex AI工作流集成
- **PrismaModule** (`src/prisma/`): PostgreSQL + pgvector 数据库层

**核心功能：**

- 基于智能体的架构，支持可配置工具集
- 复杂多智能体编排的工作流DSL
- 基于向量存储的知识库管理
- 工具探索和动态工具集注册
- 使用 ResponseBuilder 的统一 API 响应格式
- HTTP标准化的全局响应拦截器
- 基于权限的知识库访问控制

**API 响应架构：**

API 使用分层响应架构：

1. **服务层**: 返回原始数据对象
2. **控制器层**: 使用 `ResponseBuilder` 工具包装数据
3. **响应拦截器**: 处理HTTP状态码和最终格式化

所有 API 响应遵循标准格式：

```typescript
interface DataResponse<T> {
  success: true;
  data: T;
  message?: string;
  timestamp: string;
}
```

### 数据库架构

使用 PostgreSQL 配合 pgvector 扩展：

- **Agent**: 核心智能体配置，包含提示词和选项
- **Toolkit/Tool**: 模块化工具系统，支持JSON schema验证
- **Workflow**: 基于DSL的工作流定义，关联智能体
- **KnowledgeBase**: RAG向量存储，支持文件管理
- **AgentToolkit/AgentTool/AgentKnowledgeBase**: 多对多关系表

**权限架构：**

系统实现细粒度访问控制：

1. **智能体-工具集关系**: 通过 `AgentToolkit` 配置设置
2. **智能体-知识库访问**: 通过 `AgentKnowledgeBase` 管理，具有唯一约束
3. **工具权限**: 工具通过工具集关联继承智能体权限

**知识库访问控制：**

```typescript
// 权限检查使用数据库唯一约束提高效率
const hasAccess = await prisma.agentKnowledgeBase.findUnique({
  where: {
    agentId_knowledgeBaseId: { agentId, knowledgeBaseId }
  }
});
```

### Agent Web (前端)

React 应用包含：

- **React Query**: 服务端状态管理
- **React Router**: 客户端路由
- **Tailwind CSS**: 样式框架
- **Radix UI**: 组件原语

**核心页面：**

- `Dashboard.tsx`: 主概览页面
- `Agents.tsx`: 智能体管理
- `Workflows.tsx`: 工作流构建器/运行器
- `KnowledgeBases.tsx`: 知识库管理
- `Toolkits.tsx`: 工具管理
- `AgentChat.tsx`: 聊天界面

## 🔧 开发环境

### 环境要求

- Node.js >= 20
- PNPM (包管理器)
- Docker & Docker Compose

### 环境配置

1. 使用 `docker compose up --build -d` 进行容器化开发（推荐）
2. 服务运行端口：
   - Agent API: <http://localhost:3001>
   - Agent Web: <http://localhost:5173>
   - PostgreSQL: localhost:5432
   - Redis: localhost:6379

### 测试策略

**测试组织：**

- **服务测试** (`*.service.spec.ts`): 测试业务逻辑和数据操作
- **控制器测试** (`*.controller.spec.ts`): 测试HTTP层和响应格式化
- **集成测试** (`apps/agent-api/test/`): 测试完整的请求/响应周期
- **端到端测试**: 测试完整用户工作流

**运行测试：**

- `pnpm --filter agent-api test` - 运行所有单元测试
- `pnpm --filter agent-api test:e2e` - 运行端到端测试
- `pnpm --filter agent-api test <file>` - 运行特定测试文件
- `pnpm --filter agent-api test:cov` - 运行测试并生成覆盖率报告

**测试最佳实践：**

1. **服务层测试**:

   ```typescript
   // 测试原始数据返回
   expect(result).toEqual(expectedData);
   ```

2. **控制器层测试**:

   ```typescript
   // 测试 ResponseBuilder 包装的响应
   expect((result as DataResponse<any>).data).toEqual(expectedData);
   expect(result.success).toBe(true);
   ```

3. **Mock 设置**:
   - Mock 外部依赖（Prisma、HTTP客户端）
   - 使用 Jest 的类型安全 Mock
   - 在测试间重置 Mock

4. **权限测试**:
   - 测试授权和未授权访问场景
   - 验证错误响应匹配预期格式

### 代码质量

- 通过 `@workspace/eslint-config` 共享 ESLint 配置
- 通过 `@workspace/typescript-config` 共享 TypeScript 配置
- 使用 Prettier 进行代码格式化
- 提交前必须运行 `typecheck` 和 `lint`

## 🛠️ 工具集开发指南

### 创建新工具集

1. **继承 BaseToolkit**:

   ```typescript
   @toolkitId('my-toolkit-01')
   export class MyToolkit extends BaseToolkit {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gongzhaoman/hackathon-starter](https://github.com/gongzhaoman/hackathon-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
