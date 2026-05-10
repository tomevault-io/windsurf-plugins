---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 專案概述

這是一個展覽管理系統，使用 Next.js 15 (App Router) + React 19 + Prisma + PostgreSQL 構建的全端應用程式。系統包含展覽管理、作品管理、預約叫號、任務看板、前台模板編輯等功能模組。

## 開發指令

### 常用命令

```bash
# 開發環境
pnpm dev              # 啟動開發服務器 (http://localhost:3000)

# 資料庫操作
pnpm db:generate      # 生成 Prisma Client (修改 schema 後必須執行)
pnpm db:push          # 推送 Schema 到資料庫 (開發環境快速同步)
pnpm db:migrate       # 執行資料庫遷移 (生產環境用)
pnpm db:studio        # 開啟 Prisma Studio 視覺化介面
pnpm db:seed          # 填充測試資料

# 構建與程式碼檢查
pnpm build            # 構建生產版本
pnpm start            # 啟動生產服務器
pnpm lint             # ESLint 檢查

# 測試
pnpm test             # Vitest 監聽模式
pnpm test:run         # 執行單元測試 (一次性)
pnpm test:coverage    # 執行測試含覆蓋率報告
pnpm test:ui          # Vitest UI 介面
pnpm test:e2e         # Playwright E2E 測試
pnpm test:e2e:ui      # Playwright UI 模式
pnpm test:e2e:report  # 查看 E2E 測試報告
```

### 本地服務需求

開發前需確保以下服務在本地運行：
- **PostgreSQL 16**: 運行在 5432 端口
- **Redis 7**: 運行在 6379 端口
- **MinIO** (可選): 如需測試檔案儲存功能，運行在 9000 端口

## 專案架構

### 技術決策

- **前端框架**: Next.js 15 使用 App Router (不是 Pages Router)
- **UI 元件**: TailwindCSS + shadcn/ui (25 個元件，包含表單、資料展示、互動、通知等)
- **資料庫**: PostgreSQL 16 + Prisma ORM (單一 schema.prisma 包含 52 個表)
- **認證系統**: NextAuth.js v5 Beta + 自定義用戶管理
- **狀態管理**: Zustand (全局) + TanStack Query (服務端狀態)
- **套件管理**: 必須使用 pnpm (不要使用 npm 或 yarn)

### 目錄結構說明

```
src/
├── app/                    # Next.js App Router
│   ├── (auth)/            # 認證相關頁面 (登入/註冊)
│   ├── admin/             # 管理後台 (需登入)
│   │   ├── exhibitions/   # 展覽管理前端（列表、新建、編輯、詳情）
│   │   ├── users/         # 用戶管理前端（列表、新建、編輯、詳情）
│   │   ├── artworks/      # 作品管理前端（列表、新建、編輯、詳情）
│   │   ├── teams/         # 團隊管理前端（列表、新建、編輯、詳情）
│   │   ├── documents/     # 文件管理前端（列表、上傳、詳情）
│   │   ├── sponsors/      # 贊助商管理前端（列表、新建、編輯、詳情）
│   │   ├── venues/        # 場地管理前端（列表、新建、編輯、詳情）
│   │   ├── templates/     # 前台模板管理（列表、新建、編輯器）
│   │   ├── homepage-editor/ # 首頁編輯器（Wix-like 拖放編輯）
│   │   ├── settings/      # 系統設定頁面
│   │   └── page.tsx       # 後台首頁（儀表板）
│   ├── exhibitions/       # 前台展覽展示頁面
│   ├── api/               # API 路由
│   │   ├── auth/          # 認證 API
│   │   ├── exhibitions/   # 展覽 CRUD API
│   │   ├── teams/         # 團隊 CRUD API
│   │   ├── team-members/  # 團隊成員 CRUD API
│   │   ├── artworks/      # 作品 CRUD API
│   │   ├── users/         # 用戶 CRUD API
│   │   ├── sponsors/      # 贊助商 CRUD API
│   │   ├── venues/        # 場地 CRUD API
│   │   ├── documents/     # 文件 CRUD API
│   │   ├── site-templates/# 前台模板 CRUD API
│   │   ├── system-settings/# 系統設定 API
│   │   ├── upload/        # 檔案上傳 API
│   │   └── audit-logs/    # 審計日誌查詢 API
│   └── page.tsx           # 首頁
├── components/            # React 元件
│   ├── ui/                # shadcn/ui 基礎元件
│   ├── editor/            # 編輯器元件（Craft.js & Wix-like）
│   │   ├── components/    # 可編輯元件（Container, Text, Button, Image）
│   │   ├── WixLikeEditor/ # Wix-like 編輯器元件
│   │   └── settings/      # 元件屬性設定面板
│   ├── forms/             # 表單元件
│   │   ├── ExhibitionForm.tsx
│   │   ├── UserForm.tsx
│   │   ├── ArtworkForm.tsx
│   │   ├── TeamForm.tsx
│   │   ├── SponsorForm.tsx
│   │   ├── VenueForm.tsx
│   │   ├── DocumentUploadForm.tsx
│   │   └── TemplateCreateForm.tsx
│   └── layout/            # 佈局元件
│       ├── AdminSidebar.tsx
│       ├── AdminHeader.tsx
│       └── AdminLayoutClient.tsx
├── lib/                   # 核心工具
│   ├── auth.ts            # NextAuth 配置
│   ├── auth.config.ts     # NextAuth 詳細配置
│   ├── prisma.ts          # Prisma 客戶端單例
│   ├── redis.ts           # Redis 連接
│   ├── minio.ts           # MinIO 檔案儲存客戶端
│   ├── audit-log.ts       # 審計日誌工具
│   ├── api-response.ts    # 統一 API 響應格式
│   ├── api-client.ts      # 前端 API 客戶端
│   ├── scroll-animations.ts  # 滾動動畫工具 (6種效果)
│   ├── preset-templates.ts   # 預設模板配置
│   ├── utils.ts           # 通用工具函數
│   └── validations/       # Zod 驗證 schemas
│       ├── exhibition.ts
│       ├── artwork.ts
│       ├── team.ts
│       ├── sponsor.ts
│       ├── venue.ts
│       └── user.ts
├── styles/                # 全局樣式
│   └── scroll-animations.css  # 滾動動畫 CSS 定義
├── hooks/                 # Custom React Hooks
│   └── use-toast.ts       # Toast 通知 hook
├── types/                 # TypeScript 類型定義
│   └── next-auth.d.ts    # NextAuth 類型擴展
└── env.ts                 # 環境變數驗證
```

### 資料庫架構重點

1. **用戶系統**: 三種角色 (SUPER_ADMIN, CURATOR, TEAM_LEADER)
2. **展覽體系**: Exhibition → Team → TeamMember → Artwork
3. **預約系統**: ReservationSetting → ReservationSlot → Reservation
4. **任務看板**: Board → List → Card (支援拖放排序)
5. **檔案管理**: Document 表關聯多個實體，統一管理所有檔案
6. **前台模板**: SiteTemplate 儲存 Craft.js JSON，支援發布管理
7. **系統設定**: SystemSetting 鍵值對儲存，支援 JSON 資料

## 開發注意事項

### 必須遵循的規則

1. **資料庫操作**:
   - 修改 schema.prisma 後必須執行 `pnpm db:generate`
   - 絕不使用 `--accept-data-loss` 參數
   - 使用事務處理批量操作

2. **API 設計**:
   - 所有 API 路由放在 `src/app/api/` 目錄
   - 使用 Zod 驗證請求數據
   - 返回統一的錯誤格式

3. **認證授權**:
   - 使用 NextAuth 的 `auth()` 函數檢查登入狀態
   - 管理後台路由必須檢查用戶角色
   - API 路由需加入認證中間件

4. **前端開發**:
   - 使用 Server Components 優先
   - Client Components 加上 'use client' 指令
   - 表單使用 Server Actions 處理

5. **UI 元件規範**:
   - **後台管理頁面必須使用 shadcn/ui 元件**，不要使用原生 HTML 元素

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Eric920418/exhibition-system](https://github.com/Eric920418/exhibition-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
