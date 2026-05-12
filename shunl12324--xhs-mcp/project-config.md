---
trigger: always_on
description: A Model Context Protocol (MCP) server that provides tools for interacting with Xiaohongshu (小红书/RedNote). Uses Playwright for browser automation. **Version 2.0 adds multi-account support with SQLite database storage.**
---

# Xiaohongshu MCP Server

## Project Overview

A Model Context Protocol (MCP) server that provides tools for interacting with Xiaohongshu (小红书/RedNote). Uses Playwright for browser automation. **Version 2.0 adds multi-account support with SQLite database storage.**

## Tech Stack

- **Runtime**: Bun (also compatible with Node.js)
- **Language**: TypeScript (ESNext, strict mode)
- **Browser Automation**: Playwright
- **HTTP Framework**: Hono (for HTTP transport mode)
- **Database**: SQLite (better-sqlite3)
- **MCP**: @modelcontextprotocol/sdk
- **Validation**: Zod v4

## Project Structure

```
~/.xhs-mcp/                   # Data directory
├── data.db                   # SQLite database (accounts, logs, etc.)
├── logs/                     # Log files
│   └── xhs-mcp.log           # Application log
├── prompts/                  # Account prompt files (new in v2.5)
│   └── {accountName}_{accountId}/
│       ├── persona.txt       # Character definition
│       ├── select.txt        # Note selection prompt
│       └── comment.txt       # Comment generation prompt
├── temp/                     # Temporary files
│   └── images/               # Downloaded HTTP images for publishing
└── downloads/
    ├── images/{noteId}/
    └── videos/{noteId}/

src/
├── index.ts                  # Entry point with stdio/http mode switch
├── server.ts                 # MCP Server configuration and tool registration
├── http-server.ts            # HTTP transport server (StreamableHTTP)
├── core/
│   ├── config.ts             # Unified configuration with environment variables
│   ├── paths.ts              # Path utilities (re-exports from config)
│   ├── logger.ts             # Structured logging (console + file)
│   ├── account-pool.ts       # Multi-account client pool (池化管理)
│   ├── account-lock.ts       # Concurrent access prevention (互斥锁)
│   ├── multi-account.ts      # Multi-account operation helpers (并行/串行执行)
│   ├── login-session.ts      # Multi-step login session manager
│   ├── gemini.ts             # Gemini AI integration (image generation, etc.)
│   ├── explore-ai.ts         # AI decision module for explore (note selection, comment generation)
│   ├── prompt-manager.ts     # Prompt file management (read/write/render with LiquidJS)
│   ├── image-upload.ts       # Image upload utilities
│   ├── qrcode-utils.ts       # QR code generation and display utilities
│   └── prompts/              # AI prompt templates
│       └── defaults.ts       # Default prompt templates (persona, select, comment)
├── db/
│   ├── index.ts              # XhsDatabase class (组合所有 Repository)
│   ├── schema.ts             # Table definitions
│   └── repos/                # Repository 模式 - 按实体分离数据访问
│       ├── index.ts          # Repository 导出
│       ├── accounts.ts       # AccountRepository - 账户 CRUD
│       ├── profiles.ts       # ProfileRepository - 用户资料
│       ├── operations.ts     # OperationRepository - 操作日志
│       ├── published.ts      # PublishedRepository - 发布记录
│       ├── interactions.ts   # InteractionRepository - 互动记录
│       ├── downloads.ts      # DownloadRepository - 下载记录
│       ├── config.ts         # ConfigRepository - 配置键值对
│       ├── my-notes.ts       # MyNotesRepository - 我的已发布笔记缓存
│       └── explore.ts        # ExploreRepository - 探索会话和日志
├── tools/
│   ├── account.ts            # xhs_list_accounts, xhs_add_account, xhs_check_login_session, xhs_remove_account, xhs_set_account_config, xhs_get/set_account_prompt
│   ├── auth.ts               # xhs_check_auth_status (+account parameter, syncs profile)
│   ├── content.ts            # xhs_search, xhs_get_note, xhs_user_profile, xhs_list_feeds (+account parameter)
│   ├── publish.ts            # xhs_publish_content, xhs_publish_video (+account/accounts parameter)
│   ├── interaction.ts        # xhs_like_feed, xhs_favorite_feed, xhs_post_comment, xhs_reply_comment, xhs_like_comment, xhs_delete_cookies (+account/accounts)
│   ├── stats.ts              # xhs_get_account_stats, xhs_get_operation_logs
│   ├── download.ts           # xhs_download_images, xhs_download_video
│   ├── draft.ts              # xhs_create_draft, xhs_list_drafts, xhs_publish_draft, etc.
│   ├── creator.ts            # xhs_get_my_notes, xhs_query_my_notes
│   ├── notification.ts       # xhs_get_notifications
│   └── explore.ts            # xhs_explore (自动浏览)
└── xhs/
    ├── index.ts              # XhsClient facade class (supports account options)
    ├── types.ts              # TypeScript interfaces
    ├── clients/
    │   ├── browser.ts        # BrowserClient - Facade 组合所有服务
    │   ├── context.ts        # BrowserContextManager - 共享浏览器上下文
    │   ├── constants.ts      # 常量定义 (TIMEOUTS, DELAYS, SCROLL_CONFIG)
    │   └── services/         # 组合模式 - 按功能分离服务
    │       ├── index.ts      # 服务导出
    │       ├── auth.ts       # AuthService - 登录认证
    │       ├── search.ts     # SearchService - 搜索功能
    │       ├── content.ts    # ContentService - 内容获取
    │       ├── publish.ts    # PublishService - 发布功能
    │       ├── interact.ts   # InteractService - 互动功能
    │       ├── creator.ts    # CreatorService - 创作者中心

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ShunL12324/xhs-mcp](https://github.com/ShunL12324/xhs-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
