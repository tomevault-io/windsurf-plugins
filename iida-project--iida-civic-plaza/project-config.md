---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

「飯田の市民活動ひろば」- 飯田市内のNPO・市民活動を可視化するWebサイト。
Next.js 15（App Router）+ Supabase + Vercel の構成。

## 開発コマンド

```bash
# 開発サーバー起動（Turbopack使用）
npm run dev

# 本番ビルド
npm run build

# 本番サーバー起動
npm start

# ESLint実行
npm run lint
```

## 技術スタック

- **フレームワーク**: Next.js 15 (App Router, Turbopack)
- **言語**: TypeScript (strict mode)
- **スタイリング**: Tailwind CSS 3.4
- **データベース**: Supabase (PostgreSQL)
- **ストレージ**: Supabase Storage - 画像・メディア（バケット名: media）
- **UIライブラリ**: shadcn/ui, Lucide Icons
- **アニメーション**: Framer Motion
- **フォント**: M PLUS Rounded 1c (見出し/UI) + Noto Serif JP (本文) ※公開サイト
- **ホスティング**: Vercel (ISR)
- **管理画面**: 自作

## アーキテクチャ

### データフロー

```
Supabase (PostgreSQL) ←→ Next.js (公開サイト + 管理画面)
                              ↓
                    Vercel (ISR) でホスティング
```

### ルーティング構成

```
app/
├─ (frontend)/         # 公開サイト
│   ├─ page.tsx        # トップ /
│   ├─ activities/     # 活動団体紹介 /activities, /activities/[slug]
│   ├─ coming-soon/    # 準備中ページ /coming-soon
│   ├─ interviews/     # インタビュー /interviews, /interviews/[slug]
│   ├─ grants/         # 助成金情報 /grants, /grants/[slug]
│   ├─ news/           # お知らせ /news, /news/[slug]
│   ├─ faq/            # FAQ /faq
│   ├─ about/          # サイトについて /about
│   ├─ about-hiroba/   # ムトス市民活動ひろばとは
│   ├─ about-mutos/    # 「ムトス」とは
│   ├─ mutos-grants/   # ムトス飯田助成事業
│   ├─ mutos-award/    # ムトス飯田賞 歴代受賞団体
│   ├─ mutos-exchange/ # ムトス飯田学習交流会／各種講座
│   ├─ mutos-committee/# ムトス飯田推進委員会／コーディネート専門委員
│   ├─ mutos-history/  # ムトス飯田の歩み
│   ├─ mutos-logo/     # ムトス飯田ロゴマークについて
│   ├─ other-grants/   # 他団体主催の助成金事業
│   └─ other-courses/  # 他団体主催の講座情報
├─ preview/            # Coming Soonページ（独立レイアウト、Header/Footerなし）
├─ (admin)/            # 管理画面
│   └─ admin/          # /admin
└─ api/                # API Routes
```

### Coming Soonモード

`src/middleware.ts` の `COMING_SOON_MODE` フラグで制御：
- `true`: 未認証ユーザーを `/preview`（Coming Soonページ）にリダイレクト
- `false`: 通常公開
- 管理画面にログイン済み（`admin_session` Cookie）のユーザーは常にサイト閲覧可能

### ヘッダー（トグル式メニュー + お知らせティッカー）

**ヘッダー構成**: ロゴ（左）/ お知らせティッカー（中央、md+）/ メニューボタン（右）

**メニューボタン**: ピル型「メニュー⇔閉じる」ボタン（右上に4色パレットのアクセントドット付き）。常時表示されるトップレベルナビは廃止し、ボタン押下で展開する方式。

**展開挙動**:
- **xl+（1280px以上）**: 押下するとメニューボタンが消え、右端から5グループが **逆順スタガー（あらまし→助成金→事業→団体→ひろば の順）** でスプリング的にスライドイン。最終形は元の水平ヘッダーと同じ並び + 右端に × 閉じるボタン。各グループはホバーでサブメニューのドロップダウン展開。
- **xl未満**: ヘッダー下に従来通りアコーディオン式に展開（4色カラーバー付き）。

**お知らせティッカー**（中央・md+）:
- 最新の公開済み `news_posts` 1件を NEW バッジ（7日以内）＋ タイトル ＋ → で表示、`/news/[slug]` にリンク
- データは `layout.tsx` で Server Component 取得し Header に props 渡し（`revalidate=60`）
- メニュー展開中は DOMから除外。閉じ完了後 **900ms 経ってから再投入**（メニューのエグジットアニメとの同居を避けるため）

**5グループ構成**:
- ムトス市民活動ひろば → ムトス市民活動ひろばとは(/about-hiroba), 相談窓口（準備中）
- 市民活動団体紹介 → 活動団体紹介(/activities), 活動レポート(/interviews)
- ムトス飯田事業 → ムトス飯田助成事業(/mutos-grants), ムトス飯田賞(/mutos-award), 学習交流会／各種講座(/mutos-exchange)
- ムトス飯田以外の助成金・講座情報 → 他団体主催の助成金事業(/other-grants), 他団体主催の講座情報(/other-courses)
- ムトス飯田事業のあらまし → 「ムトス」とは(/about-mutos), 推進委員会／コーディネート専門委員(/mutos-committee), ムトス飯田の歩み(/mutos-history), ロゴマークについて(/mutos-logo)

※「一般社団法人ムトス飯田市民ファンド」はサブメニュー配置先未定のためコメントアウト中

未実装の「相談窓口」のみ `/coming-soon` にリダイレクト。

## パスエイリアス

```typescript
// tsconfig.json
"@/*" → "./src/*"
```

## 環境変数

必要な環境変数（`.env.local`に設定）:
- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- `SUPABASE_SERVICE_ROLE_KEY`
- `NEXT_PUBLIC_SITE_URL`
- `GEMINI_API_KEY` - Google Gemini API（AI要約機能用）
- `CRON_SECRET` - Vercel Cron 認証用トークン（Supabaseスリープ防止 keepalive）

## Supabase 設定

- **プロジェクトID**: gxsvyzvaalwywnylakgu
- **リージョン**: ap-northeast-1 (東京)
- **Storageバケット**: media（公開）

### 作成済みテーブル

**メインテーブル（RLS有効）:**
- `activity_categories` - 活動分野マスター（10件投入済み）
- `activity_areas` - 活動エリアマスター（18件投入済み）
- `tags` - タグマスター
- `organizations` - 市民活動団体
- `interviews` - ロングインタビュー
- `grants` - 助成金情報
- `news_posts` - お知らせ
- `faqs` - よくある質問

**中間テーブル（RLS有効）:**
- `organization_categories` - 団体 × 活動分野
- `organization_areas` - 団体 × 活動エリア
- `organization_tags` - 団体 × タグ
- `grant_categories` - 助成金 × 対象分野

### RLSポリシー
- 公開データ（`is_published = true`）は誰でもSELECT可能
- マスターテーブルは全データSELECT可能
- INSERT/UPDATE/DELETE は暗黙的に拒否（ポリシーなし = 拒否）
- 管理画面は `service_role` で RLS をバイパス
- Storage `media` バケットは公開URL直接アクセスのみ（`storage.objects` の SELECT ポリシーは設置せず、リスティング不可）

### NOT NULL 制約

以下のカラムは NOT NULL を設定済み（NULL による予期せぬ挙動を防ぐ）：
- 全メインテーブルの `is_published`, `is_featured`, `is_recruiting`, `created_at`, `updated_at`
- マスター/FAQ の `sort_order`
- マスターの `created_at`

### マスターテーブルのUNIQUE制約

`activity_categories.name`, `activity_areas.name`, `tags.name` に UNIQUE 制約あり（`slug` に加えて `name` も重複不可）。

### 外部キーの ON DELETE 挙動

| 子テーブル | 親テーブル | 挙動 | 理由 |
|----------|-----------|------|------|
| `interviews.organization_id` | `organizations.id` | `SET NULL` | 団体削除時も編集記事として残す |
| 中間テーブル各種 | 各親 | `CASCADE` | 関連が消えたら紐付けも削除 |

### データベースインデックス

FKカラムおよび頻出クエリパターンにインデックスを設定済み：

| インデックス名 | テーブル | カラム | 種別 |
|--------------|---------|--------|------|
| `idx_interviews_organization_id` | interviews | organization_id | FK |
| `idx_organization_categories_category_id` | organization_categories | category_id | FK（逆方向） |
| `idx_organization_areas_area_id` | organization_areas | area_id | FK（逆方向） |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iida-project/iida-civic-plaza](https://github.com/iida-project/iida-civic-plaza) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
