---
trigger: always_on
description: 本文件为 Claude Code (claude.ai/code) 在此仓库中工作时提供指导。
---

# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 在此仓库中工作时提供指导。

## 常用命令

### 前端
- **开发服务器**: `cd EduNexus && npm run dev`
- **构建**: `cd EduNexus && npm run build`（执行 vue-tsc + vite build）
- **预览**: `cd EduNexus && npm run preview`
- **类型检查**: `cd EduNexus && npx vue-tsc --noEmit`

### 后端
- **conda**: `conda activate edunexus`
- **启动服务**: `cd backend && uvicorn app.main:app --reload --port 8001`
- **初始化数据库**: `cd backend && python -m app.seed`
- **Docker (PostgreSQL + Redis)**: `cd backend && docker-compose up -d`

## 项目概述

**EduNexus** 是一个 Vue 3 + TypeScript + Vite 的多智能体 AI 教育平台，采用暗色主题、Google AI Studio 风格 UI。平台实现了**五大核心模块**——多个专业 AI 智能体协同工作，完成从学生画像构建、内容生成、学习路径规划、资源推送到智能辅导与评估的完整教育闭环。

**当前阶段**: v1.0.0 前后端已全面完成，用于竞赛实机演示。v2.0.0 下一代 Agent 引擎在 `agents/` 目录中隔离开发中，目标是从 Prompt 驱动升级为具备 Tool Use、Memory、自主规划能力的类 Claude Code 智能体系统。

> **版本策略**: v1.0.0 代码保持稳定不变，v2.0.0 开发仅限 `backend/app/agents/` 和 `EduNexus/src/agents/` 目录。

---

## 前端技术栈

- **框架**: Vue 3.5.32（Composition API，`<script setup lang="ts">`）
- **构建**: Vite 8.0.10 + vue-tsc 3.2.7 + TypeScript 6.0.2
- **路由**: vue-router 4.6.4（createWebHistory 模式）
- **样式**: Tailwind CSS 3.4.19 + autoprefixer 10.5.0 + postcss 8.5.13，纯暗色主题，CSS 变量
- **UI 组件**: shadcn-vue（基于 reka-ui 2.9.7、class-variance-authority 0.7.1、clsx 2.1.1、tailwind-merge 3.5.0）
- **图标**: lucide-vue-next 1.0.0
- **工具库**: @vueuse/core 14.3.0、@tanstack/vue-table 8.21.3
- **状态管理**: Pinia 3.0.4（authStore、profileStore、factoryStore、learningStore、deliveryStore、tutorStore、assessmentStore）
- **HTTP 客户端**: axios 1.16.0（`src/api/client.ts`，自动注入 JWT Token + Token 自动刷新 + 统一错误处理）
- **推荐 IDE 扩展**: Vue.volar（不要使用 Vetur）

## 后端技术栈

- **运行时**: Python 3.12+ / FastAPI（原生 async/await）
- **数据库**: PostgreSQL（pgvector 扩展）+ SQLAlchemy async
- **AI/LLM**: Google Gemini API（gemma-3-27b / gemini-2.5-flash）
- **实时通信**: FastAPI 内置 WebSocket
- **认证**: JWT（python-jose）+ bcrypt 密码哈希
- **部署**: Docker Compose（PostgreSQL:5433 + Redis:6379）

---

## 前端目录结构

```
EduNexus/
├── src/
│   ├── main.ts                          # 应用入口（createApp + Pinia + Router）
│   ├── App.vue                          # 根组件（仅含 <router-view />）
│   ├── style.css                        # Tailwind 指令 + 全局暗色样式 + Inter 字体
│   ├── env.d.ts                         # .vue 模块类型声明
│   ├── router/index.ts                  # 路由定义（7 条路由，/tutor 和 /assessment 未注册）
│   ├── api/                             # ★ API 客户端层（已实现）
│   │   ├── client.ts                    # axios 实例 + JWT 拦截器
│   │   ├── auth.ts                      # login(), register()
│   │   ├── profile.ts                   # submitOnboarding(), getMyProfile()
│   │   ├── factory.ts                   # generateContent(), getTaskStatus(), getTaskResources(), chatWithAgent()
│   │   ├── graph.ts                     # planPath(), getGraph(), getPersonalizedPath(), getNodeResources()
│   │   ├── resources.ts                 # getTodayResources(), getResourceDetail(), updateProgress(), getTrending()
│   │   └── websocket.ts                 # EduNexusWebSocket 类（自动重连 + 心跳 + 事件分发）
│   ├── types/                           # ★ TypeScript 类型定义（已实现）
│   │   ├── api.ts                       # ApiError, PaginatedResponse<T>
│   │   ├── auth.ts                      # User, LoginRequest, RegisterRequest, AuthResponse
│   │   ├── profile.ts                   # OnboardingSubmission, ProfileDimensions, ProfileResponse
│   │   ├── factory.ts                   # GenerateRequest, TaskStatusResponse, GeneratedResource, AgentChatRequest/Response, AgentLogEntry
│   │   ├── graph.ts                     # KnowledgeNode, KnowledgeEdge, PathPlanRequest/Response, KnowledgeGraphData
│   │   └── resources.ts                 # TodayResource, ResourceProgress, TrendingItem, ResourceDetail
│   ├── stores/                          # ★ Pinia 状态管理（已实现）
│   │   ├── authStore.ts                 # token/user/isLoggedIn/isOnboarded + localStorage 持久化
│   │   ├── profileStore.ts              # profileData/version + fetchProfile()
│   │   ├── factoryStore.ts              # 生成任务状态 + WebSocket 实时事件 + 轮询兜底 + 资源加载
│   │   ├── learningStore.ts             # 图谱节点/边 + 阶段分组 + plan()
│   │   └── deliveryStore.ts             # 今日资源 + 热点 + fetchAll() + updateResourceProgress()
│   ├── layouts/
│   │   └── DashboardLayout.vue          # 主布局（SidebarProvider + 侧边栏 + 顶栏 + 内容区）
│   ├── components/
│   │   ├── AppSidebar.vue               # 侧边栏导航（6 个菜单项 + 个人信息弹窗 + 退出登录）
│   │   ├── HelloWorld.vue               # 遗留的 Vite 脚手架文件（未使用，可删除）
│   │   └── ui/                          # shadcn-vue 组件库（~23 个组件系列）
│   │       ├── alert/ avatar/ badge/ breadcrumb/ button/ card/
│   │       ├── dialog/ dropdown-menu/ input/ label/ popover/
│   │       ├── process/                 # ★ 新增：Process 进度组件
│   │       ├── progress/ scroll-area/ separator/ sheet/
│   │       ├── sidebar/ skeleton/ tabs/ tooltip/
│   ├── views/
│   │   ├── LoginView.vue                # 登录页
│   │   ├── RegisterView.vue             # 注册页
│   │   ├── OnboardingView.vue           # 破冰对话式画像构建
│   │   ├── HomeView.vue                 # 仪表盘首页
│   │   ├── ContentFactory/              # 模块二：多智能体内容生成工作台
│   │   │   ├── index.vue                # 工作台主控
│   │   │   └── components/
│   │   │       ├── TopicInput.vue       # 主题关键词输入
│   │   │       ├── AgentStatusCard.vue  # 智能体状态卡片
│   │   │       ├── AgentTerminal.vue    # 日志终端

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Elk-123/EduNexus](https://github.com/Elk-123/EduNexus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
