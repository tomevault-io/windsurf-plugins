---
trigger: always_on
description: 本文件为 AI 编码助手提供仓库级上下文与开发约定。
---

# Noesis（智枢）- Agent 项目指南

本文件为 AI 编码助手提供仓库级上下文与开发约定。

## 项目概述

Noesis（智枢）是一个基于 Vue 3 + FastAPI + LangGraph 的全栈 AI 对话应用，包含前端 Web 部分和后端服务部分。

### 前端 (frontend/)
- 基于 Vue 3 + Vite 6 + TypeScript + Naive UI
- 支持多种大模型（Spark、SiliconFlow、Ollama、OpenAI-compatible）的流式输出响应
- 问答类型：智能问答、故障运维、测试用例生成、深度研究

### 后端 (backend/)
- 基于 FastAPI + LangGraph + `create_noesis_agent` 统一工厂
- 数据库：MySQL
- 向量库：Qdrant
- LLM：阿里云 DashScope (Qwen 系列)
- ORM：SQLAlchemy (异步)
- 认证：JWT

### 核心功能
1. **通用智能问答** - 基于 `create_noesis_agent` + RAG 搜索工具
2. **故障运维问答** - 基于 `create_noesis_agent` + MCP 运维工具
3. **测试用例生成** - 基于 LangGraph StateGraph 自定义 workflow
4. **深度研究问答** - 基于 `create_noesis_agent` + 文件系统/Skills

## 常用命令

### 前端
```bash
# 安装依赖
pnpm i

# 本地开发（http://localhost:2048）
pnpm dev

# 构建生产版本
pnpm build

# GitHub Pages 部署（使用 hash 路由）
pnpm build:gh-pages

# 代码检查
pnpm lint

# 代码检查并自动修复
pnpm lint:fix

# Stylelint 检查
pnpm stylelint
```

### 后端
```bash
# 验证代码改动（检查进程能否正常拉起）
uv run app.py
```

## 核心架构

### 前端架构

#### 应用入口
- `frontend/src/main.ts` - 应用入口，注册插件、路由、状态管理
- `frontend/src/App.vue` - 根组件

#### 路由系统
- `frontend/src/router/index.ts` - 路由配置，路由模式根据 `isMockDevelopment` 切换（hash/history）
- `frontend/src/router/routes.ts` - 根路由定义
- `frontend/src/router/child-routes.ts` - 子路由（ChatRoot、McpChat 等）
- `frontend/src/router/permission.ts` - 路由守卫，处理认证

#### 状态管理（Pinia）
- `frontend/src/store/index.ts` - Store 初始化
- `frontend/src/store/business/userStore.ts` - 用户认证状态（token、登录/登出）
- `frontend/src/store/business/index.ts` - 业务状态（qa_type、file_list、task_id）和 AI 对话流式处理
- `frontend/src/store/business/initChatHistory.ts` - 聊天历史初始化
- `frontend/src/store/hooks/useAppStore.ts` - 应用状态 Hook

#### SSE 流式处理
- `frontend/src/views/chat/useSSEStream.ts` - SSE 流式响应处理 Hook
- `frontend/src/views/chat/messageParts.ts` - 消息部件处理

#### API 层
- `frontend/src/api/index.ts` - API 入口
- `frontend/src/api/chat.ts` - 聊天相关 API
- `frontend/src/api/client.ts` - API 客户端配置
- `frontend/src/api/knowledgeBase.ts` - 知识库 API
- `frontend/src/api/skills.ts` - Skills 文件目录 API（磁盘）
- 使用原生 Fetch API，通过 `userStore.getUserToken()` 获取认证 token

#### Hooks
- `frontend/src/hooks/useClipText.ts` - 剪贴文本
- `frontend/src/hooks/useCopyCode.ts` - 代码复制
- `frontend/src/hooks/useCurrentInstance.ts` - 当前实例
- `frontend/src/hooks/useTheme.ts` - 主题切换

#### 视图层
- `frontend/src/views/chat.vue` - **核心对话页面**（60KB+，包含聊天逻辑）
- `frontend/src/views/chat/` - 聊天子模块
  - `index.vue` - Markdown 渲染组件
  - `useSSEStream.ts` - SSE 流式处理 Hook
  - `messageParts.ts` - 消息部件
- `frontend/src/views/Login.vue` - 登录页面
- `frontend/src/views/mcp/MCPClient.vue` - MCP 客户端页面
- `frontend/src/views/knowledge-base/` - 知识库相关页面
  - `KnowledgeBase.vue` - 知识库主页
  - `CollectionDetail.vue` - 集合详情
- `frontend/src/views/skills/` - Skills 文件目录管理
  - `SkillsManagement.vue` - 目录树与预览
- `frontend/src/views/TestAssistant.vue` - 测试助手
- `frontend/src/views/SuggestedPage.vue` - 建议页面
- `frontend/src/views/PdfViewer.vue` - PDF 查看器
- `frontend/src/views/FileUploadManager.vue` - 文件上传管理
- `frontend/src/views/TableModal.vue` - 表格弹窗
- `frontend/src/views/DefaultPage.vue` - 默认页面

#### 组件库
- `frontend/src/components/MarkdownPreview/` - Markdown 渲染核心
  - `index.vue` - 主组件，处理流式响应
  - `plugins/` - 插件（highlight, markdown, preWrapper）
- `frontend/src/components/Layout/` - 布局组件
  - `default.vue` - 默认布局
  - `SidearPage.vue` - 侧边栏页面
  - `SlotArea.vue` / `SlotFrame.vue` / `SlotCenterPanel.vue` - 插槽布局
- `frontend/src/components/Navigation/` - 导航组件
  - `NavBar.vue` - 导航栏
  - `NavSideBar.vue` - 侧边导航
  - `NavFooter.vue` - 页脚
  - `NavOctocat.vue` - Octocat 导航
  - `SideBar.vue` - 侧边栏
- `frontend/src/components/KnowledgeBase/` - 知识库组件
  - `DocumentDrawer.vue` - 文档抽屉
  - `ShardDetail.vue` - 分片详情
- `frontend/src/components/TodoList/` - Todo 列表组件
- `frontend/src/components/FileUploadManager/` - 文件上传管理
- `frontend/src/components/TableList/` - 表格列表
- `frontend/src/components/Pagination/` - 分页组件
- `frontend/src/components/IconFont/` - 图标字体
- `frontend/src/components/IconifyIcon/` - Iconify 图标
- `frontend/src/components/ClipBoard/` - 剪贴板
- `frontend/src/components/AssistantReplyToolbar/` - Assistant 回复工具栏
- `frontend/src/components/ReasoningBlock/` - 推理过程展示块
- `frontend/src/components/ToolCallCollapse/` - 工具调用折叠组件
- `frontend/src/components/404.vue` - 404 页面

#### 配置
- `frontend/src/config/env.ts` - **关键配置**：`isMockDevelopment` 控制模拟/真实 API 模式
  - 开发环境默认 `true`（使用模拟数据）
  - 设置为 `false` 时调用真实大模型接口

### 后端架构

```
backend/
├── api/                    # FastAPI 路由层
│   ├── __init__.py         # 导出所有 router
│   ├── chat_api.py         # 聊天历史 API
│   ├── login_api.py        # 登录接口
│   ├── user_api.py         # 用户接口
│   ├── knowledge_base_api.py # 知识库 API
│   ├── skill_api.py        # Skills 文件目录 API（磁盘）
├── services/               # 业务逻辑层
│   ├── chat_service.py     # 聊天历史服务
│   ├── qa_service.py       # 问答服务
│   ├── login_service.py
│   ├── user_service.py
│   ├── skill_fs_service.py # Skills 磁盘目录扫描与 ZIP 解压
│   └── qdrant_service.py   # 向量库服务
├── schemas/                 # Pydantic 请求/响应模型
│   ├── chat_vo.py
│   ├── login_vo.py
│   ├── qa_vo.py
│   ├── skill_vo.py
│   └── knowledge_base_schema.py
├── model/                   # SQLAlchemy ORM 模型
│   ├── db_models.py        # 通用模型
│   └── chat_models.py      # 聊天会话/消息模型
├── llm/                     # LLM 集成（MODEL_TYPE 工厂）
│   └── factory.py          # get_llm()

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sheep1433/Noesis](https://github.com/Sheep1433/Noesis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-08 -->
