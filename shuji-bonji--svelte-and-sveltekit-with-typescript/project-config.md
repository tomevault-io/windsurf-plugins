---
trigger: always_on
description: 日本語によるTypeScript中心のSvelte 5/SvelteKit完全マスター学習コンテンツ。
---

# CLAUDE.md - TypeScriptで学ぶ Svelte5/SvelteKit 学習ガイド

## プロジェクト概要

日本語によるTypeScript中心のSvelte 5/SvelteKit完全マスター学習コンテンツ。

- **リポジトリ**: https://github.com/shuji-bonji/Svelte-and-SvelteKit-with-TypeScript
- **公開URL**: https://shuji-bonji.github.io/Svelte-and-SvelteKit-with-TypeScript/
- **ビルドツール**: SvelteKit + mdsvex（SveltePressは廃止済み）

### 対象読者

- TypeScript/JavaScriptでのWeb開発経験者
- SPA/WebAPI開発経験があるがSSR/SSGは不慣れなエンジニア
- SvelteKitでビジネスレベルのサービスを構築したい人
- Spring Boot、ASP.NET Core、Angular等の経験を活かしたいエンタープライズ開発者

## 技術スタック

| 技術 | バージョン | 備考 |
|------|-----------|------|
| Svelte | 5.x以上 | Runesシステム必須 |
| SvelteKit | 2.x以上 | |
| TypeScript | 6.x以上 | strictモード必須 |
| Vite | 8.x以上 | |
| Node.js | 20.x LTS以上 | 推奨: 20.x LTS |

## コーディング規約

**Skill化済み**: `.claude/skills/svelte5-coding-standards/SKILL.md` を参照。

主要ルール:
- Svelte 5 Runes（$state, $derived, $effect, $props）のみ使用
- レガシー構文（$:、export let、`<slot />`、on:event、createEventDispatcher）は禁止
- TypeScript strictモード、`any`禁止
- `$app/state` 推奨（`$app/stores` はレガシー）
- `PageProps` / `LayoutProps` 推奨（`PageData` / `LayoutData` の直接使用はレガシー）

## Svelte MCP の利用（必須）

**このプロジェクトのコード修正・記事更新時は、必ず [Svelte MCP](https://svelte.dev/docs/mcp) を利用すること。**

- **`svelte-autofixer`**: Svelteコンポーネントやコード例を書いた/修正した際に必ず実行し、レガシーパターンがないか検証する
- **`get-documentation`**: Svelte 5 / SvelteKit の最新仕様を確認する際に使用する
- **`list-sections`**: 関連するドキュメントセクションを特定する際に使用する

記事内のコード例（Markdownのコードブロック）も、主要なものは `svelte-autofixer` に通して検証すること。

## カリキュラム構成

### 第1部：入門編 (`/introduction/`)
環境構築、Hello World、TypeScript設定、学習パス、CLIツール

### 第2部：Svelte編 (`/svelte/`)

```
svelte/
├── basics/          # Svelteの基本（コンポーネント、テンプレート構文、ライフサイクル、アクション、トランジション、TypeScript統合）
├── runes/           # Runesシステム（$state, $derived, $effect, $props, $bindable, $host, $inspect, 比較）
├── advanced/        # 実践編（ストア、クラス、組み込みクラス、Snippets、コンポーネントパターン、TypeScriptパターン、await expressions、hydratable、reactivity-window）
└── architecture/    # アーキテクチャ（SPA+API、BaaS=Firebase/Supabase、GraphQL、既存システム統合、デスクトップ・モバイル=Tauri/Electron/Capacitor）
                     #  ※ hybrid-integration はスタブ、それ以外は実装済み
```

### 第3部：SvelteKit編 (`/sveltekit/`)

```
sveltekit/
├── basics/          # 基礎編（概要、プロジェクト構造、ファイルシステム、レンダリング戦略、app.d.ts、$appモジュール）
├── routing/         # ルーティング（基本、動的、高度、shallow、link-options）
├── data-loading/    # データ取得（Load関数、TypeScript型、データフロー、SPA/無効化、ストリーミング、戦略）
├── architecture/    # アーキテクチャ詳解（レンダリング戦略/パイプライン、ハイドレーション、アクセスログ、データロード、ルーティング内部）
├── server/          # サーバーサイド（フォーム処理、WebSocket/SSE、Remote Functions、server-only-modules）
├── application/     # アプリケーション構築（認証、セッション、テスト、状態管理、snapshots）
├── optimization/    # 最適化（ビルド、キャッシュ、PWA、observability）
└── deployment/      # デプロイ（プラットフォーム、実行環境、packaging）
```

### 第4部：実装例 (`/examples/`)
ブログシステム（基礎/Markdown）、TODOアプリ、認証システム（Cookie/JWT）、データフェッチング、WebSocket、Mermaid SSRデモ

### 第5部：リファレンス (`/reference/`)
Svelte 5完全リファレンス、SvelteKit 2.x完全リファレンス

### 第6部：技術詳解 (`/deep-dive/`)
コンパイル時最適化、リアクティブ状態、派生値比較、HTMLテンプレート、素のJS構文、自動生成型、Web Components/CSS戦略、プレースホルダー

## ディレクトリ構造

```
Svelte-and-SvelteKit-with-TypeScript/
├── src/
│   ├── routes/              # ページとレイアウト
│   ├── lib/
│   │   ├── components/      # 再利用可能コンポーネント
│   │   │                    #   - AutoPageNavigation（前後ページナビ）
│   │   │                    #   - TableOfContents（右サイドバー目次）
│   │   │                    #   - Sidebar / Navbar
│   │   │                    #   - LiveCode（svelte.dev/playground iframe 埋め込み）
│   │   │                    #   - Mermaid（SSR 対応の Mermaid 描画）
│   │   │                    #   - SeoMeta（OGP/Twitter Card メタ集約）
│   │   │                    #   - PwaUpdatePrompt（PWA 新版検知＆更新通知 UI）
│   │   │                    #   - Admonition（:::note などの情報強調）
│   │   ├── config/
│   │   │   └── sidebar.ts   # サイドバー構造の単一情報源
│   │   ├── stores/          # グローバルストア(.svelte.ts)
│   │   └── utils/           # ユーティリティ（navigation-from-config.ts、playground-url.ts 等）
│   ├── app.html             # HTML シェル＋favicon/OGP/PWA メタ
│   └── app.d.ts             # ambient 型（virtual:pwa-register 等）
├── static/                  # 静的ファイル（favicon 一式、og-image.png、icon-*.png、robots.txt）
├── markdown-plugins/        # mdsvex 前段で走る Svelte preprocessor 群
│   └── preprocess-admonition-import.js  # ::: ディレクティブ → <Admonition> 変換＋import 自動注入
├── .claude/
│   └── skills/
│       └── svelte5-coding-standards/  # コーディング規約Skill
├── vite.config.js           # Vite 設定（@vite-pwa/sveltekit で PWA 生成）
├── svelte.config.js         # mdsvex 設定・LiveCode コードブロック変換ロジック・admonitionPreprocessor 登録
├── tsconfig.json
├── CLAUDE.md                # このファイル
└── CHANGELOG.md             # 変更履歴詳細
```

## コンテンツ作成ガイドライン

### Markdownファイル構造

```markdown
---
title: ページタイトル（35 文字以内）
description: 120-160文字。「TypeScript」「Svelte5」を必ず含める。具体的な学習内容を列挙。
---

導入文：ページの目的と概要

## 基本的な使い方
（TypeScriptコード例）

## 実践例

## よくある間違い

## まとめ

## 次のステップ
```

**重要**: フロントマター直後の `# {title}` は不要（`+layout.svelte` 側でfrontmatterの`title`から自動レンダリング）。

#### title の文字数制約（SEO）

`SeoMeta.svelte` が `<title>{title} | TypeScriptで学ぶ Svelte 5/SvelteKit</title>` の形でサフィックス（35 文字）を自動付与する。Bing Webmaster Tools の警告（70 文字超過）を回避するため、**frontmatter `title` は 35 文字以内**に収めること。

サフィックスに `TypeScript` / `Svelte 5` / `SvelteKit` の 3 キーワードが既に含まれるため、ページ title では：
- これらの繰り返し（例：`Svelte 5 の ...`、`TypeScript で ...`、`SvelteKit Remote Functions`）を避ける

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shuji-bonji/Svelte-and-SvelteKit-with-TypeScript](https://github.com/shuji-bonji/Svelte-and-SvelteKit-with-TypeScript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
