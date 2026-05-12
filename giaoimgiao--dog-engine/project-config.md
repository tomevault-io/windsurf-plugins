---
trigger: always_on
description: **项目名称**: dog写作引擎 (dog-Engine)
---

# dog写作引擎 - 架构分析文档

## 📋 项目概览

**项目名称**: dog写作引擎 (dog-Engine)  
**定位**: 面向网文作者与编辑团队的开源创作与阅读一体化引擎  
**技术栈**: Next.js 15 + TypeScript + Gemini AI + Tailwind CSS  
**核心特性**: 前端直调AI、在线书城、创作管理、AI率检测、可扩展服务端能力

---

## 🏗️ 系统架构

### 架构风格
- **混合架构**: 前后端分离 + 前端直调AI
- **渐进式增强**: 核心功能纯前端运行，服务端功能可选启用
- **微服务思想**: 书源解析、AI代理、图片反代等能力模块化

### 技术栈详解

#### 前端框架
- **Next.js 15.3.3**: App Router + SSR/CSR混合渲染
- **React 18.3.1**: UI组件库基础
- **TypeScript 5**: 类型安全保障

#### UI/UX层
- **Tailwind CSS 3.4**: 原子化CSS框架
- **shadcn/ui**: 基于Radix UI的组件库
  - 30+个UI组件 (Dialog, Accordion, Tabs等)
  - 完全可定制主题系统
- **Lucide React**: 图标库
- **设计理念**: 
  - 淡黄色主色调 (#FAF4C0) - 仿羊皮纸质感
  - Literata serif字体 - 复古文学感
  - 移动端优先设计

#### AI能力层
- **Gemini AI**: Google生成式AI
  - 前端直调 (Browser → Google API)
  - 可选服务端代理 (Genkit)
- **@genkit-ai**: AI flow编排框架
- **支持模型**: 
  - `gemini-2.5-flash` (快速响应)
  - `gemini-2.5-pro` (高质量输出)
  - 可动态列举用户可用模型

#### 数据存储
- **LocalStorage**: 主存储方案
  - 书籍数据 (`books-v1`)
  - AI配置 (`gemini-api-key`, `gemini-model`)
  - 书源配置 (`book-sources-v2`, `book-source-auth`)
- **Firebase** (可选): 社区功能持久化

#### 网络层
- **fetch API**: 原生HTTP客户端
- **https-proxy-agent**: 服务端代理支持
- **Cheerio**: HTML解析 (书源内容提取)
- **VM2**: 安全沙箱执行用户书源JS代码

---

## 📁 项目结构分析

```
dog-Engine/
├── src/
│   ├── app/                    # Next.js App Router页面
│   │   ├── page.tsx           # 首页 (功能导航中心)
│   │   ├── layout.tsx         # 全局布局
│   │   ├── api/               # API路由 (服务端逻辑)
│   │   │   ├── bookstore/     # 书城相关API
│   │   │   ├── community/     # 社区功能API
│   │   │   └── test-proxy/    # 代理测试
│   │   ├── books/[bookId]/    # 书籍详情 (动态路由)
│   │   ├── bookstore/         # 在线书城页面
│   │   ├── community/         # 创作社区
│   │   ├── review/            # 网文审稿
│   │   ├── settings/          # 书源管理
│   │   └── talent-test/       # 网文天赋测试
│   │
│   ├── components/            # React组件库
│   │   ├── Editor.tsx         # 核心编辑器组件 (1084行)
│   │   ├── ChapterManager.tsx # 章节管理
│   │   ├── WorldBookManager.tsx # 世界设定管理
│   │   ├── CharacterCardManager.tsx # 角色卡片管理
│   │   ├── DeconstructOutline.tsx # 细纲拆解
│   │   ├── GeminiSettings.tsx # AI配置UI
│   │   ├── AiDetector.tsx     # AI率检测
│   │   └── ui/                # shadcn/ui组件 (40+个)
│   │
│   ├── lib/                   # 核心业务逻辑库
│   │   ├── gemini-client.ts   # 前端直调Gemini (498行)
│   │   ├── book-source-utils.ts # 书源解析引擎 (1067行)
│   │   ├── book-source-rule-parser.ts # 规则解析器
│   │   ├── book-source-storage.ts # 书源存储
│   │   ├── jsonpath-parser.ts # JSON路径解析
│   │   ├── proxy-fetch.ts     # 代理工具
│   │   ├── types.ts           # TypeScript类型定义
│   │   └── actions/           # Server Actions
│   │
│   ├── ai/                    # AI能力模块
│   │   ├── genkit.ts          # Genkit配置
│   │   ├── dev.ts             # 开发入口
│   │   └── flows/             # AI流程定义
│   │       ├── generate-story-chapter.ts
│   │       ├── respond-to-prompt-in-role.ts
│   │       ├── review-manuscript.ts
│   │       ├── refine-chapter-with-world-info.ts
│   │       └── list-models.ts
│   │
│   ├── hooks/                 # React Hooks
│   │   ├── use-toast.ts       # Toast通知
│   │   ├── use-mobile.tsx     # 移动端检测
│   │   └── useLocalStorage.ts # LocalStorage Hook
│   │
│   └── data/
│       └── community-prompts.json # 社区提示词库
│
├── docs/                      # 项目文档
│   ├── blueprint.md           # 产品蓝图
│   ├── frontend-ai-guide.md   # 前端AI使用指南
│   ├── frontend-migration-summary.md
│   └── proxy-setup.md         # 代理配置文档
│
├── book_sources.json          # 书源配置文件
├── book_source_auth.json      # 书源认证配置
├── apphosting.yaml            # Firebase部署配置
└── next.config.ts             # Next.js配置
```

---

## 🔧 核心功能模块

### 1. AI写作助手 (Editor.tsx)

**职责**: 智能续写、改写、风格迁移、角色扮演

**核心能力**:
- 🤖 **角色扮演回复**: 基于角色卡片和世界观上下文
- 📝 **智能续写**: 支持连续对话，保留上下文
- 🎨 **风格迁移**: 可调节温度、输出长度
- 🔄 **上下文压缩**: 多章节内容自动压缩为剧情清单
- 🧠 **思维链**: 可选包含AI思考过程 (thinking budget)

**技术实现**:
```typescript
// 前端直调Gemini API
import { generateContent, generateContentStream } from '@/lib/gemini-client';

// 构建消息历史
const messages = [
  { role: 'user', parts: [{ text: systemPrompt }] },
  { role: 'model', parts: [{ text: '明白' }] },
  ...conversationHistory
];

// 流式生成
for await (const chunk of generateContentStream({
  contents: messages,
  model: selectedModel,
  config: { temperature, maxOutputTokens }
})) {
  // 实时追加到编辑器
}
```

**用户配置项** (LocalStorage):
- `gemini-api-key`: API密钥
- `gemini-model`: 默认模型
- `gemini-temperature`: 生成温度
- `gemini-max-tokens`: 最大输出长度
- `gemini-safety`: 安全过滤级别
- `gemini-debug`: 调试模式
- `gemini-timeout-ms`: 请求超时
- `gemini-retries`: 重试次数

---

### 2. 在线书城 (bookstore/)

**职责**: 多书源聚合、搜索、分类、阅读、导入

**书源解析架构**:
```
用户输入书源规则 (JSON)
      ↓
book-source-rule-parser.ts (解析CSS/JS/JSONPath混合规则)
      ↓
VM2沙箱安全执行 (隔离用户代码)
      ↓
Cheerio解析HTML
      ↓
提取结构化数据
      ↓
返回 BookstoreBook / BookstoreChapter
```

**支持的规则类型**:
- **CSS选择器**: `@css:.book-item@text`
- **JS代码**: `<js>document.title</js>`
- **JSON路径**: `$.data.books[*].title`
- **混合规则**: `@css:.cover@src##@js:baseUrl+result`
- **占位符**: `{{page}}`, `{{host()}}`, `{{source.xxx}}`
- **正则替换**: `replaceRegex`, `sourceRegex`

**关键文件**:
- `src/lib/book-source-utils.ts` (1067行): 
  - `fetchSearchResults()` - 搜索书籍
  - `fetchBookDetail()` - 获取详情
  - `fetchChapterContent()` - 获取章节内容
  - `parseRuleWithCssJs()` - 规则解析核心

**书源配置示例**:
```json
{
  "name": "示例书源",
  "url": "https://example.com",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giaoimgiao/dog-Engine](https://github.com/giaoimgiao/dog-Engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
