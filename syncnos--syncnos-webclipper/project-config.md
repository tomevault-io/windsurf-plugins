---
trigger: always_on
description: 本仓库以 WebClipper（浏览器扩展）为活动主线；Apple 客户端与 CLI 已拆分至独立仓库：
---

# 仓库指南

本仓库以 WebClipper（浏览器扩展）为活动主线；Apple 客户端与 CLI 已拆分至独立仓库：

- iOS: `SyncNos/SyncNos-Health`
- macOS: `SyncNos/SyncNos-booknotes`
- CLI: `SyncNos/SyncNos-CLI`

## 项目结构

```
SyncNos-Webclipper/
├── src/                         # WebClipper 逻辑与 UI
├── public/                      # 静态资源
├── tests/                       # 单测
├── cloudflare-workers/          # Notion OAuth worker
├── .github/                     # 开发指南、deepwiki 与 CI
└── README.md
```

## 构建与运行

```bash
npm install              # 安装 WebClipper 依赖
npm run dev              # 启动 WebClipper Chrome 开发模式
npm run build            # 构建 WebClipper Chrome 产物
npm run compile          # TypeScript 编译检查
npm run test             # 单元测试
npm run check            # 产物校验
```

## 核心规范

| 规范               | 路径                                   |
| ------------------ | -------------------------------------- |
| 仓库总览           | `README.md`                            |
| deepwiki 总入口    | `.github/deepwiki/INDEX.md`            |
| deepwiki 业务入口  | `.github/deepwiki/business-context.md` |
| 本文档（统一入口） | `AGENTS.md`                            |
| WebClipper UI 规范 | `src/ui/AGENTS.md`                     |

## 开发工作流

### 先判断产品线

- 开始任何仓库级理解、规划、评审或文档改动前，先读 `.github/deepwiki/business-context.md`；需要继续展开时，再从 `.github/deepwiki/INDEX.md` 进入对应专题页。
- 一般情况下不需要全量通读 `.github/deepwiki/`；按 `.github/deepwiki/INDEX.md` 导航到最小必要页面即可。
- 修改扩展代码时，优先查看 `.github/deepwiki/modules/webclipper.md` 与本文档（`AGENTS.md`），先确认变更属于 `background`、`content`、`popup` 还是 `app`。
- iOS/macOS 相关内容已迁移到独立仓库，本仓库不再承载 Apple 客户端代码更新。
- 若改动影响共享业务说明或仓库级入口文档，代码确认后同步更新相关 `AGENTS.md`、`.github/deepwiki/INDEX.md`、`.github/deepwiki/business-context.md` 与 `README.md`。

### WebClipper 工作流

1. 先确认职责边界：采集逻辑放 `collectors/content`，持久化与路由放 `background`，界面交互放 `popup` / `app`。
2. 权限、content scripts、消息协议、构建产物变更要同时检查 `wxt` 入口、manifest 结果与 CI 脚本是否一致。
3. Settings / Conversations UI 改动时，不要只改单个组件；同时核对这些真源：
   - `src/viewmodels/settings/types.ts`（section 分组与顺序）
   - `src/viewmodels/settings/useSettingsSceneController.ts`（`inpage_display_mode` / `ai_chat_auto_save_enabled` / `ai_chat_dollar_mention_enabled` / `ai_chat_cache_images_enabled` / `web_article_cache_images_enabled` / `chat_with_*`）
   - `src/ui/settings/sections/InpageSection.tsx`（`markdown_reading_profile_v1` / `anti_hotlink_rules_v1`）
   - `src/viewmodels/settings/insight-stats.ts`（Insight 统计口径）
   - `src/services/integrations/chatwith/chatwith-settings.ts`（Chat with AI 模板 / 平台 / 截断）
   - `src/services/integrations/item-mention/**`（`$ mention`：站点门控、候选搜索、插入载荷构建）
   - `src/ui/styles/tokens.css`（主题 token：仅跟随 `prefers-color-scheme`）
   - `src/ui/shared/SelectMenu.tsx`（source/site 筛选菜单的 `adaptiveMaxHeight` 与可视区域计算）
   - `src/ui/popup/PopupShell.tsx` / `src/ui/app/AppShell.tsx`（列表统计跳转 Insight 的路由入口）
   - `src/ui/conversations/ConversationListPane.tsx` / `pending-open.ts`（来源筛选持久化与窄屏 detail bridge）
   - `src/viewmodels/conversations/conversations-context.tsx` / `DetailHeaderActionBar.tsx` / `DetailNavigationHeader.tsx`（详情头 `open / tools` 槽位分发与窄屏一致性；`tools` 仅用于本地工具动作（例如 cache-images），不包含 Chat with）
   - `src/ui/comments/react/ThreadedCommentsPanel.tsx` / `src/services/integrations/chatwith/chatwith-comments-header-actions.ts`（Chat with 仅归属评论侧栏：sidebar header + comment-level）
4. 默认验证顺序使用：
   - `npm run compile`
   - `npm run test`
   - `npm run build`
5. 若改动涉及 Firefox、发布打包、manifest/content script 重写或产物完整性，再补：
   - `npm run build:firefox`
   - `npm run check`
6. 发布包与 AMO Source 包由 GitHub Actions 和 `.github/scripts/webclipper/*.mjs` 负责，本地以开发验证为主。
7. 如果改动的是视频字幕采集，再额外核对 `src/services/bootstrap/video-transcript-capture.ts`、`src/services/bootstrap/video-transcript-capture-content-handlers.ts`、`src/entrypoints/video-transcript-*.content.ts`、`src/collectors/video/**`、`src/platform/context-menus/clipper-context-menu.ts`、`src/services/protocols/conversation-kinds.ts` 与 `src/ui/settings/sections/VideosSection.tsx`。

## WebClipper 分层与契约（必须遵守）

### 项目结构（分层）

- `src/ui/**`：UI（popup/app/inpage），只放组件/样式/DOM 面板；设置页由 `src/ui/settings/SettingsScene.tsx` 统筹；Inpage 相关设置集中在 `src/ui/settings/sections/InpageSection.tsx`；视频字幕采集入口集中在 `src/ui/settings/sections/VideosSection.tsx`
- `src/viewmodels/**`：ViewModel（hooks/context），只做 UI 状态编排与调用 service
- `src/services/**`：Service（用例/业务流程），承接平台交互与业务逻辑
- `src/platform/**`：平台适配（runtime/ports/storage/webext 等）
- `src/collectors/**`：站点采集规则（content-side DOM 解析）与视频字幕采集（YouTube / Bilibili subtitle bridge）

### 分层与依赖方向（强约束）

- 依赖方向：`ui -> viewmodels -> services -> (platform, domain, client, sync, shared, ...)`
- 禁止反向依赖：`services` 不得 import `ui/viewmodels`
- 禁止平台直连：`ui/**` 与 `viewmodels/**` 不得 import `platform/**`
- 业务与 UI 解耦：可复用业务算法/数据处理必须下沉到 `services/**`（或更底层的 domain/client 模块）

边界自检（手动）：

- `rg -n "src/platform|/platform/" src/ui`
- `rg -n "src/platform|/platform/" src/viewmodels`

### TypeScript 路径别名（约定）

- `@ui/*` -> `src/ui/*`
- `@viewmodels/*` -> `src/viewmodels/*`
- `@services/*` -> `src/services/*`
- `@platform/*` -> `src/platform/*`
- `@collectors/*` -> `src/collectors/*`
- `@entrypoints/*` -> `src/entrypoints/*`
- `@i18n/*` -> `src/ui/i18n/*`

自检（手动）：

- `rg -n "@platform/" src/ui src/viewmodels`

### UI 圆角规范（Concentric Radius）

- 圆角 token 真源：`src/ui/styles/tokens.css`（`--radius-outer/card/control/chip/inline/pill`）。
- UI 规范细则：`src/ui/AGENTS.md` 的 `B2.2 · 同心圆角分级`。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SyncNos/SyncNos-Webclipper](https://github.com/SyncNos/SyncNos-Webclipper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
