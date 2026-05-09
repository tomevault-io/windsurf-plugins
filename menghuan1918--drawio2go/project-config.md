---
trigger: always_on
description: 基于 Electron + Next.js + HeroUI 构建的跨平台 DrawIO 编辑器应用。
---

# DrawIO2Go - AI 代理开发指南

## 项目概述

基于 Electron + Next.js + HeroUI 构建的跨平台 DrawIO 编辑器应用。

### 核心技术栈

- **前端框架**: Next.js 15 (App Router) + React 19
- **UI 库**: HeroUI v3 (v3.0.0-beta.1) - 基于 React Aria Components 的复合组件模式
- **样式**: Tailwind CSS v4 (⚠️ 必须 v4，v3 不兼容)
- **AI 集成**: AI SDK v6 (Vercel AI SDK)
- **DrawIO 集成**: 原生 iframe 实现
- **桌面应用**: Electron 38.x
- **语言**: TypeScript
- **主题**: 现代扁平化设计，Material Design风格 (#3388BB 蓝色主题)

### HeroUI v3 核心特性

- **语义化设计**: 使用 primary/secondary/tertiary 替代 solid/flat/bordered
- **可访问性优先**: 基于 React Aria Components，内置 WCAG 2.1 AA 合规性
- **复合组件模式**: 灵活组合、深度自定义，而非扁平化 props
- **GPU 加速动画**: 原生 CSS 动画替代 Framer Motion，性能更优
- **树摇优化**: 仅打包使用的组件，减小包体积
- **AI 友好**: 为 AI 辅助开发设计的 API 和文档结构
- **完全类型安全**: TypeScript 全覆盖，IntelliSense 支持

### 设计系统规范

> 详细文档请参考 `app/styles/AGENTS.md`

**设计令牌（Design Tokens）**:

- **圆角**: 4px (小) / 8px (标准) / 12px (大)
- **间距**: 4px 基准（4/8/16/24/32px）
- **阴影**: Material Design 4 层阴影系统
- **动画**: 150ms (快) / 200ms (标准) / 300ms (慢)

**核心原则**:

- ✅ 使用 CSS 变量 (`var(--radius)`, `var(--shadow-2)`)
- ✅ 扁平化设计，避免渐变和复杂效果
- ✅ 简单交互反馈，避免干扰性动画（脉冲、浮动）
- ❌ 禁止硬编码颜色、尺寸和阴影值

### 项目结构

```
app/
├── components/         # React 组件库 [详细文档 → app/components/AGENTS.md]
│   ├── DrawioEditorNative.tsx    # DrawIO 编辑器（原生 iframe + PostMessage）
│   ├── TopBar.tsx                # 顶栏组件（含国际化）
│   ├── UnifiedSidebar.tsx        # 统一侧边栏容器
│   ├── SettingsSidebar.tsx       # 设置侧边栏
│   ├── ChatSidebar.tsx           # 聊天侧边栏主组件（AI SDK v6）
│   ├── VersionSidebar.tsx        # 版本侧边栏主组件
│   ├── chat/                     # 聊天组件模块化架构（17个子组件）
│   │   ├── Composer.tsx          # 输入区域（含 Skill 按钮、页面选择器）
│   │   └── AGENTS.md             # 聊天模块详细文档
│   ├── settings/                 # 设置相关子组件（含国际化）
│   │   └── AGENTS.md             # 设置模块详细文档
│   ├── toast/                    # Toast 通知组件系统
│   │   └── AGENTS.md             # Toast 模块详细文档
│   ├── version/                  # 版本管理子组件
│   │   ├── AGENTS.md             # 版本模块详细文档
│   │   ├── VersionCard.tsx       # 版本卡片（折叠式）
│   │   ├── VersionTimeline.tsx   # 版本时间线
│   │   ├── VersionCompare.tsx    # 版本对比全屏弹层
│   │   ├── CreateVersionDialog.tsx # 创建版本对话框
│   │   ├── PageSVGViewer.tsx     # 多页 SVG 查看器
│   │   └── diff-engine/          # 差异计算引擎模块
│   └── AGENTS.md                 # 组件库完整文档
├── i18n/               # 国际化配置 [详细文档 → app/i18n/AGENTS.md]
│   ├── config.ts                 # i18n 配置文件
│   ├── client.ts                 # i18next 初始化（动态加载 JSON）
│   └── hooks.ts                  # 类型安全 i18n Hooks
├── lib/                # 工具库 [详细文档 → app/lib/AGENTS.md]
│   ├── drawio-tools.ts          # DrawIO XML 操作工具集
│   ├── drawio-xml-utils.ts      # DrawIO XML 归一化与解压工具
│   ├── frontend-tools.ts        # 前端工具执行（DrawIO read/edit_batch，顺序执行）
│   ├── svg-export-utils.ts      # 多页面 SVG 导出工具
│   ├── svg-smart-diff.ts        # SVG 智能差异对比引擎
│   ├── config-utils.ts          # LLM 配置规范化工具（含 Skill 配置）
│   ├── compression-utils.ts     # 压缩/解压工具（pako）
│   ├── version-utils.ts         # 版本号解析与排序工具
│   ├── utils.ts                 # 通用工具函数
│   ├── storage/                 # 统一存储抽象层 [详细文档 → app/lib/storage/AGENTS.md]
│   │   ├── adapter.ts           # 存储适配器抽象类
│   │   ├── indexeddb-storage.ts # IndexedDB 实现（Web）
│   │   ├── sqlite-storage.ts    # SQLite 实现（Electron）
│   │   ├── storage-factory.ts   # 存储实例工厂
│   │   ├── current-project.ts   # 当前工程 ID 持久化工具
│   │   ├── xml-version-engine.ts # XML 版本恢复引擎（Diff 重放）
│   │   ├── page-metadata.ts     # 页面元数据提取工具
│   │   ├── page-metadata-validators.ts # 元数据校验（页面数、SVG 体积等）
│   │   ├── constants.ts         # 常量定义（WIP_VERSION 等）
│   │   ├── constants-shared.js  # 跨环境共享常量
│   │   ├── default-diagram-xml.js # 默认空白图表 XML
│   │   ├── types.ts             # 存储层类型定义
│   │   ├── AGENTS.md            # 存储层详细文档
│   │   └── index.ts             # 统一导出
│   └── AGENTS.md                 # 工具库完整文档
├── types/              # 类型定义 [详细文档 → app/types/AGENTS.md]
│   ├── chat.ts                  # 聊天相关类型
│   ├── drawio-tools.ts          # DrawIO 工具类型
│   ├── socket.ts                # 工具调用基础类型（与传输层解耦）
│   └── global.d.ts              # 全局类型声明
├── hooks/              # React Hooks [详细文档 → app/hooks/AGENTS.md]
│   ├── useStorageSettings.ts    # 设置持久化 Hook
│   ├── useStorageProjects.ts    # 项目管理 Hook
│   ├── useCurrentProject.ts     # 当前工程管理 Hook（超时保护 + 自动兜底）
│   ├── useStorageConversations.ts   # 会话管理 Hook
│   ├── useStorageXMLVersions.ts     # XML 版本管理 Hook
│   ├── useVersionCompare.ts     # 版本对比状态管理 Hook
│   └── useDrawioEditor.ts       # DrawIO 编辑器操作封装 Hook
├── api/                # API 路由
│   ├── ai-proxy/                # 纯 AI 代理端点（仅转发，不含业务逻辑）
│   ├── health/                  # 健康检查（在线心跳）
│   └── test/                    # 测试 API 路由
├── config/              # 配置文件
│   └── skill-elements.json      # 绘图技能配置（主题、知识库元素）
├── styles/             # 模块化样式系统 [详细文档 → app/styles/AGENTS.md]
│   ├── base/                    # 基础样式（reset、变量）
│   ├── components/              # 组件样式
│   ├── layout/                  # 布局样式
│   ├── themes/                  # 主题样式
│   └── utilities/               # 工具样式
├── page.tsx            # 主页面
├── layout.tsx          # 根布局（含国际化初始化）
└── globals.css         # 全局样式入口

public/
└── locales/            # 翻译资源（en-US, zh-CN，按需扩展 ja-JP）

electron/               # 桌面应用 [详细文档 → electron/AGENTS.md]
server.js              # Next.js 自定义 HTTP 服务器
```

## 开发准则


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Menghuan1918/drawio2go](https://github.com/Menghuan1918/drawio2go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
