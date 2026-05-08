---
trigger: always_on
description: このドキュメントは、cc-recommender プロジェクトの開発ガイドラインを定義します。
---

# cc-recommender Development Guide

このドキュメントは、cc-recommender プロジェクトの開発ガイドラインを定義します。
新しいコードを追加・変更する際は、必ずこのガイドラインに従ってください。

## プロジェクト概要

Claude Code のスキル、プラグイン、MCP サーバーを推薦する MCP サーバー。
プロジェクトを分析し、適切な拡張機能を提案します。

## アーキテクチャ原則

### 1. レイヤードアーキテクチャ

```
presentation (tools) → business logic (services) → data access (repositories)
                    ↓
                 config, utils, types
```

**依存関係のルール:**
- 上位レイヤーは下位レイヤーに依存できる
- 下位レイヤーは上位レイヤーに依存してはいけない
- 同一レイヤー内での依存は最小限に

### 2. 単一責任の原則 (SRP)

- 各ファイルは1つの責務のみを持つ
- 大きくなりすぎた場合は分割を検討

### 3. 直接インポートの原則

**原則: 直接個別ファイルからインポート**

**例外: 以下のディレクトリのみ index.ts を使用可能**
- `types/index.ts` - 型定義の公開 API（頻繁に使用されるため）
- `tools/handlers/index.ts` - MCP ツールの公開 API（外部インターフェース）

```typescript
// ✅ 許可: 公開 API からのインポート
import { Recommendation, ProjectInfo } from "../../types/index.js";
import { recommendSkills, searchSkills } from "../../tools/handlers/index.js";

// ✅ 推奨: その他は直接個別ファイルからインポート
import { analyzeProject } from "../../services/analyzer/project-analyzer.service.js";
import { calculateScore } from "../../services/recommender/scoring/scorer.js";

// ❌ 禁止: services/, config/, utils/ などに index.ts を作成
import { analyzeProject } from "../../services/analyzer/index.js"; // NG
```

**理由:**
- インポート元が明確
- 循環依存の防止
- 未使用コードの検出が容易
- 外部 API は index.ts で明示化
- 内部実装は直接インポートで変更に強く

## ディレクトリ構造

```
src/
├── config/              # 設定ファイル（定数、マッピング）
│   ├── constants.ts     # アプリケーション定数
│   ├── file-mappings.ts # ファイル拡張子とフレームワークのマッピング
│   ├── curated-list-sources.ts # キュレーションリストソース定義
│   ├── direct-skill-sources.ts # 直接スキルソース定義
│   ├── env.ts           # 環境変数設定
│   └── scoring-config.ts # スコアリングの重み・閾値
│
├── repositories/        # データアクセス層
│   ├── recommendation.repository.ts # データベース読み込み・キャッシュ
│   └── remote-data.repository.ts    # リモートデータ取得（CDN/GitHub）
│
├── utils/              # 共通ユーティリティ
│   └── glob-matcher.ts # Glob パターンマッチング
│
├── types/              # 型定義（ドメイン別に分割）
│   ├── index.ts        # 型定義の公開 API
│   ├── domain-types.ts # ドメイン型（Recommendation, Author, etc.）
│   ├── service-types.ts # サービス型（ProjectInfo, ScoredRecommendation）
│   └── raw-types.ts    # 外部データ型（RawPluginEntry, etc.）
│
├── schemas/            # Zod バリデーションスキーマ
│   └── tool-schemas.ts # MCP ツールの入力スキーマ
│
├── services/           # ビジネスロジック層
│   ├── analyzer/       # プロジェクト分析サービス
│   │   ├── parsers/    # 言語別パーサー
│   │   │   ├── package-json.parser.ts
│   │   │   ├── requirements-txt.parser.ts
│   │   │   ├── go-mod.parser.ts
│   │   │   └── （その他言語用パーサー）
│   │   └── project-analyzer.service.ts # メイン分析ロジック
│   │
│   ├── fetchers/       # 外部データ取得
│   │   ├── mcp/        # MCP サーバー取得
│   │   │   ├── mcp-fetcher.ts
│   │   │   └── official-mcp-fetcher.ts
│   │   ├── plugins/    # プラグイン取得
│   │   │   └── plugin-fetcher.ts
│   │   └── skills/     # スキル取得
│   │       ├── common/  # 共通ユーティリティ
│   │       │   ├── github-api.ts # GitHub API 呼び出し
│   │       │   ├── skill-parser.ts # スキルパース処理
│   │       │   └── types.ts # フェッチャー用の内部型
│   │       ├── curated-list-fetcher.ts # キュレーションリスト取得
│   │       ├── direct-skill-fetcher.ts # 直接スキル取得
│   │       └── skill-fetcher.ts # メインフェッチャー
│   │
│   ├── recommender/    # 推薦サービス
│   │   ├── scoring/    # スコアリングロジック
│   │   │   └── scorer.ts
│   │   ├── recommendation.service.ts # メイン推薦ロジック
│   │   ├── search.service.ts         # 検索機能
│   │   ├── quality-scorer.ts         # 品質スコア算出
│   │   └── formatters.ts             # 結果フォーマッター
│   │
│   └── security-scanner.service.ts # セキュリティスコア取得
│
├── tools/              # MCP ツール層（プレゼンテーション）
│   └── handlers/       # 各ツールの実装
│       ├── index.ts    # ツールの公開 API
│       ├── recommend-skills.tool.ts
│       ├── search-skills.tool.ts
│       ├── get-skill-details.tool.ts
│       ├── list-categories.tool.ts
│       └── get-stats.tool.ts
│
├── server/             # MCP サーバー設定
│   ├── mcp-server.ts   # サーバーセットアップ
│   └── tool-registry.ts # ツール登録
│
└── index.ts            # エントリーポイント（最小限）
```

### Claude Code 拡張機能ディレクトリ

```
.claude/
├── agents/              # カスタムエージェント定義
│   ├── architecture-agent.md        # アーキテクチャチェック
│   ├── security-agent.md            # セキュリティチェック
│   ├── typescript-agent.md          # TypeScript ベストプラクティス
│   ├── documentation-agent.md       # ドキュメント整合性チェック
│   └── dependabot-review-agent.md   # Dependabot PRレビュー
│
├── commands/            # カスタムコマンド
│   ├── pre-commit-check.md    # コミット前の包括的チェック
│   └── dependabot-review.md   # Dependabot PRの分析・レビュー
│
└── skills/              # カスタムスキル
    ├── documentation-check.md  # ドキュメントチェックスキル
    ├── architecture-check.md   # アーキテクチャチェックスキル
    ├── security-check.md       # セキュリティチェックスキル
    ├── typescript-check.md     # TypeScript チェックスキル
    ├── tdd.md                  # TDD スキル
    └── dependabot-review.md    # Dependabot PRレビュースキル
```

**各エージェントの役割:**
- **Architecture Agent**: プロジェクトのアーキテクチャと設計思想を確認し、一貫性を保つ
- **Security Agent**: セキュリティベストプラクティスを確認し、脆弱性を検出
- **TypeScript Agent**: TypeScript のベストプラクティスに従ってコードをレビュー
- **Documentation Agent**: ドキュメントとコードの整合性を確認
- **Dependabot Review Agent**: Dependabot PRを分析し、リスク評価とレビューコメントを投稿

**カスタムコマンド:**
- `/pre-commit-check`: コミット前に4つのエージェント（Architecture, Security, TypeScript, Documentation）を並列実行し、包括的なコード品質チェックを実施
- `/dependabot-review`: Dependabot PRを分析してリスク評価・レビューコメントを投稿（PR番号指定可、未指定時は全Dependabot PRを並列分析）

## ファイル命名規則

### パターン

```
<name>.<type>.ts
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yuji0809/cc-recommender](https://github.com/yuji0809/cc-recommender) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
