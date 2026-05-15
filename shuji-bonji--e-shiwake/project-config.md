---
trigger: always_on
description: Generates a Svelte Playground link with the provided code.
---

You are able to use the Svelte MCP server, where you have access to comprehensive Svelte 5 and SvelteKit documentation. Here's how to use the available tools effectively:

## Available MCP Tools:

### 1. list-sections

Use this FIRST to discover all available documentation sections. Returns a structured list with titles, use_cases, and paths.
When asked about Svelte or SvelteKit topics, ALWAYS use this tool at the start of the chat to find relevant sections.

### 2. get-documentation

Retrieves full documentation content for specific sections. Accepts single or multiple sections.
After calling the list-sections tool, you MUST analyze the returned documentation sections (especially the use_cases field) and then use the get-documentation tool to fetch ALL documentation sections that are relevant for the user's task.

### 3. svelte-autofixer

Analyzes Svelte code and returns issues and suggestions.
You MUST use this tool whenever writing Svelte code before sending it to the user. Keep calling it until no issues or suggestions are returned.

### 4. playground-link

Generates a Svelte Playground link with the provided code.
After completing the code, ask the user if they want a playground link. Only call this tool after user confirmation and NEVER if code was written to files in their project.

---

# e-shiwake 開発ガイド

## Svelte MCP Server

Svelte MCP server を利用可能。Svelte 5 と SvelteKit のドキュメントにアクセスできる。

### 利用可能なツール

1. **list-sections** - ドキュメントセクション一覧を取得（最初に実行）
2. **get-documentation** - 特定セクションのドキュメントを取得
3. **svelte-autofixer** - Svelte コードの問題を分析・修正提案（コード作成時は必ず使用）
4. **playground-link** - Svelte Playground リンクを生成

## アプリ概要

**e-shiwake**（電子仕訳）— フリーランス・個人事業主向けの仕訳入力 + 証憑管理 PWA

- **ローカルファースト**: サーバー不要、IndexedDB にデータ保存
- **SPA モード**: SvelteKit の SSR は使わない
- **PWA**: Service Worker でオフライン動作
- **iPad 対応**: File System Access API が使えないため、IndexedDB + エクスポートで対応

**URL**: `https://shuji-bonji.github.io/e-shiwake/`

> **機能詳細・データモデル・複式簿記ルール・勘定科目コード体系** については
> `src/routes/llms.txt/+server.ts` または各ヘルプページの `content.md` を参照のこと。
>
> **各ページの機能仕様を確認したい場合**: 該当ページの `content.md` が正（Single Source of Truth）。
> 例: 消費税集計の仕様 → `src/routes/help/tax-category/content.md`

## 技術スタック

- **フレームワーク**: SvelteKit
- **言語**: TypeScript
- **UI**: shadcn-svelte + Tailwind CSS v4
- **データ保存**: IndexedDB（Dexie.js）
- **ファイル操作**: File System Access API（デスクトップ）
- **PWA**: @vite-pwa/sveltekit + Workbox

## サイトマップ

```
/                           # 仕訳帳（ホーム）
├── /ledger                 # 総勘定元帳
├── /trial-balance          # 試算表
├── /profit-loss            # 損益計算書
├── /balance-sheet          # 貸借対照表
├── /tax-summary            # 消費税集計
├── /fixed-assets           # 固定資産台帳
├── /blue-return            # 青色申告決算書
├── /reports                # 帳簿出力（一括印刷・CSV ZIP）
├── /archive                # 検索機能付アーカイブ（年度締め・長期保存）
├── /invoice                # 請求書一覧
├── /invoice/[id]           # 請求書編集
├── /vendors                # 取引先管理
├── /accounts               # 勘定科目管理
├── /data                   # データ管理（バックアップ/エクスポート/インポート）
├── /settings               # 設定（事業者情報・証憑保存・容量）
├── /export                 # エクスポート（レガシー）
└── /help                   # ヘルプ トップ
    ├── /getting-started    # はじめに
    ├── /journal            # 仕訳入力
    ├── /ledger             # 総勘定元帳
    ├── /trial-balance      # 試算表
    ├── /tax-category       # 消費税区分
    ├── /evidence           # 証憑管理
    ├── /accounts           # 勘定科目
    ├── /fixed-assets       # 固定資産台帳
    ├── /blue-return        # 青色申告決算書
    ├── /invoice            # 請求書
    ├── /data-management    # 設定
    ├── /backup-restore     # バックアップ・リストア
    ├── /import-export      # インポート・エクスポート
    ├── /archive            # 検索機能付アーカイブ保存
    ├── /pwa                # PWA・インストール
    ├── /shortcuts          # キーボードショートカット
    ├── /glossary           # 用語集
    └── /webmcp             # WebMCP（AIエージェント連携）
```

## 既知の課題

### ストレージモードとデータ移行

証憑の保存先として2つのモード（`filesystem` / `indexeddb`）がある。

| シナリオ                               | 問題点                             |
| -------------------------------------- | ---------------------------------- |
| Chrome(filesystem) → Safari(indexeddb) | `filePath`のみでファイル実体がない |
| Safari(indexeddb) → Chrome(filesystem) | blobがなく、ファイルも存在しない   |
| 端末移行時                             | 証憑PDFが引き継がれない            |

**現状の対応**: JSONエクスポートは仕訳データのみ。ZIPエクスポートで証憑同梱。

## 開発フェーズ

- Phase 1: MVP ✅
- Phase 1.5: PWA & UX強化 ✅
- Phase 2: 帳簿機能 ✅
- Phase 3: 確定申告対応 ✅
- Phase 3.5: バックアップ・アーカイブ改善 — 進行中（`docs/design/backup-archive-redesign.md` 参照）
- Phase 4: 国際展開（i18n・多通貨） — 未着手

## コーディング規約

### ファイル構成

```
src/
├── lib/
│   ├── components/     # 再利用可能なコンポーネント
│   │   ├── ui/         # shadcn-svelte コンポーネント
│   │   ├── layout/     # AppSidebar, AppHeader
│   │   ├── journal/    # JournalEntry, JournalLine, PdfDropZone, SearchHelp
│   │   ├── blue-return/ # BlueReturnSettingsDialog 等
│   │   ├── data/       # ExportCard, ImportCard, CapacityCard, BusinessInfoCard
│   │   ├── invoice/    # InvoicePrint 等
│   │   └── help/       # HelpSection, HelpTable, HelpNote
│   ├── hooks/          # useJournalPage 等のカスタムフック
│   ├── stores/         # Svelte stores（fiscalYear.svelte.ts）
│   ├── db/             # IndexedDB 関連（Dexie、リポジトリパターン）
│   ├── types/          # TypeScript 型定義
│   └── utils/          # ユーティリティ関数
├── routes/             # SvelteKit ルート（サイトマップ参照）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shuji-bonji/e-shiwake](https://github.com/shuji-bonji/e-shiwake) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
