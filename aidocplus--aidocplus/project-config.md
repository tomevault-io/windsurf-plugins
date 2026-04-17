---
trigger: always_on
description: AI 编程助手指南，供 Claude / Windsurf 等工具在此仓库工作时参考。
---

# CLAUDE.md

AI 编程助手指南，供 Claude / Windsurf 等工具在此仓库工作时参考。

**始终用中文与用户对话。界面文字、提示信息全部使用中文。字体为宋体，大小 16，跨平台兼容。**

---

## 项目概览

**AiDocPlus** — 跨平台 AI 文档桌面编辑器（官网：https://aidocplus.com）

| 层  | 技术                                                                 |
| --- | -------------------------------------------------------------------- |
| 后端 | Rust + Tauri 2.x                                                    |
| 前端 | React 19 + TypeScript 5.9 + Zustand + Radix UI + Tailwind CSS 4    |
| 编辑器 | CodeMirror 6                                                       |
| 构建 | Vite 8 + Turborepo + pnpm                                          |
| 国际化 | i18next                                                            |

当前版本：**0.3.13**

---

## 目录结构

```
AiDocPlus/
├── apps/desktop/
│   ├── src-tauri/                  # Rust 后端
│   │   ├── src/
│   │   │   ├── main.rs
│   │   │   ├── commands/           # IPC 命令（32 个文件）
│   │   │   ├── ai.rs / api_gateway.rs / api_server.rs
│   │   │   ├── document.rs / template.rs / plugin.rs
│   │   │   └── native_export/ pandoc.rs ...
│   │   └── bundled-resources/      # 构建产物（.gitignore）
│   │       ├── ai-providers/
│   │       ├── document-templates/
│   │       └── prompt-templates/
│   └── src-ui/
│       ├── index.html / manager.html  # Vite 多入口
│       └── src/
│           ├── document-types/     # 11 种内置文档类型
│           ├── doctype-sdk/        # DocType SDK（registry/host/types/hooks）
│           ├── components/         # editor/ chat/ coding/ tabs/ ...
│           ├── plugins/            # 插件系统（28 个插件 + _framework/）
│           ├── stores/             # useAppStore useSettingsStore useTemplatesStore
│           ├── manager/            # 资源管理器前端
│           └── i18n/               # zh/en 翻译文件
├── packages/
│   ├── shared-types/               # 共享 TypeScript 类型
│   ├── manager-rust/               # 资源管理器 Rust crate（35+ 命令）
│   ├── manager-shared/             # 资源管理器共享类型
│   ├── manager-ui/                 # 资源管理器 React 组件库
│   ├── mcp-server/                 # MCP Server（23 个工具）
│   ├── sdk-python/                 # Python SDK
│   ├── sdk-js/                     # JS SDK
│   └── utils/
├── resources/                      # 资源源数据（prompt-templates/ ai-providers/ doc-templates/）
└── scripts/                        # build-resources.sh release.sh 等
```

---

## 常用命令

```bash
# 首次初始化
bash scripts/build-resources.sh   # 生成 bundled-resources + shared-types/generated/
pnpm install

# 开发
cd apps/desktop && pnpm tauri dev

# 编译检查
cd apps/desktop/src-ui && npx tsc --noEmit
cd apps/desktop/src-tauri && cargo check

# 发布构建（macOS）
cd apps/desktop && pnpm tauri build --target aarch64-apple-darwin

# 端口占用
lsof -ti:1420 | xargs kill -9
```

Tauri dev 模式：Rust 文件修改自动重编译重启，前端 Vite 热更新。

---

## 核心功能

- **多标签页编辑**：每个标签页独立面板状态（AI 消息、流式状态）
- **五大工作区**：生成区 / 内容区 / 合并区 / 功能区 / 编程区
- **AI 内容生成 & 聊天**：流式输出、停止控制、附件参考、联网搜索、深度思考（CoT）
- **多模态 AI**：聊天支持上传/粘贴图片（≤5 张，自动适配 OpenAI Vision / Anthropic）
- **文档级 AI 服务绑定**：每个文档可独立绑定不同 AI 服务，通过 `getAIInvokeParamsForService()` 解析
- **版本控制**：自动保存，预览恢复
- **多格式导出**：Markdown / HTML / DOCX / TXT（原生 + Pandoc）
- **附件系统**：参考文件，AI 生成时自动读取
- **插件系统**：28 个插件，自注册 + 自动发现 + manifest 驱动
- **编程区**：多语言代码编辑执行（Python/JS/TS/HTML 等），AI 自动化
- **开放 API**：HTTP JSON-RPC（11 命名空间 30+ 操作）+ MCP Server（23 工具）+ Python/JS SDK
- **文档标签与收藏**：标签筛选、`_starred` 内部标签
- **资源管理器**：多窗口（Tauri WebviewWindow），管理提示词模板、AI 提供商、文档模板
- **长篇小说**：设定集、章节层级、AI 四层记忆写作、伏笔追踪、全书导出+EPUB

---

## 文档类型系统（11 种）

### SDK 架构

- **入口**：`src-ui/src/doctype-sdk/`（types.ts / registry.ts / host.ts / hooks.ts）
- **实现**：`src-ui/src/document-types/{type-id}/`
- **共享基础**：`src-ui/src/document-types/_shared/`

### 内置文档类型

| 类型 ID | 名称 | 布局 | 编辑器 / 工作区 |
|---|---|---|---|
| `normal` | 通用文档 | standard | EditorPanel + ChatPanel |
| `study-notes` | 学习体会 | standard | DocTypeEditorBase + StudyNotesAISidebar |
| `novel` | 长篇小说 | full | NovelDocWorkspace（三栏）+ NovelAISidebar |
| `translation` | 中英翻译 | standard | TranslationWorkspace（双栏）+ TranslationAISidebar |
| `diary` | 日记 | full | DiaryDocWorkspace + DiaryAISidebar |
| `essay` | 散文写作 | full | EssayDocWorkspace + EssayAISidebar |
| `stock-research` | 股票研究 | full | StockResearchWorkspace + StockResearchAISidebar |
| `imitative-writing` | 仿写练习 | full | ImitativeWritingWorkspace + ImitativeWritingAISidebar |
| `calculator` | 计算文档 | full | CalculatorWorkspace + CalculatorAISidebar |
| `task-list` | 任务清单 | full | TaskListWorkspace + TaskListAISidebar |
| `outline` | 大纲写作 | full | OutlineWorkspace + OutlineAISidebar |

> **规则**：所有文档类型（包括 full 布局）必须有右侧 AI 聊天面板。

### 共享基础组件

- `DocTypeAIChatBase` — 统一 AI 聊天框架（流式/联网/深度思考/消息操作）
- `DocTypeChatMessage` — 消息渲染（MarkdownPreview + think 标签 + 复制）
- `DocTypeEditorBase` — 编辑器包装（自动保存 + AI 插入监听）
- `styles.ts` — Design Tokens 常量

### Design Tokens（统一规范）

| 元素 | 类名 |
|---|---|
| 快捷按钮 | `h-7 text-xs px-2 gap-1` |
| 消息气泡 | `rounded-lg p-3 text-sm` |
| 用户消息 | `bg-primary/10 ml-8` |
| AI 消息 | `bg-muted mr-2` + MarkdownPreview |
| 状态栏 | `px-3 py-1 border-t text-xs text-muted-foreground bg-card` |
| 工具栏 | `px-3 py-1.5 border-b bg-card` |

### 新建文档类型流程

1. `document-types/{type-id}/definition.ts`（实现 `DocTypeDefinition`）
2. `{TypeName}Editor.tsx` / `{TypeName}AISidebar.tsx`
3. 在 `register.ts` 中注册
4. 添加 i18n 键

---

## 插件系统（28 个插件）

### 操作位置规则（强制）

| 场景 | 位置 |
|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AiDocPlus) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
