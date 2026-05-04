---
trigger: always_on
description: > **Note**: `CLAUDE.md` は `AGENTS.md` へのシンボリックリンクです。ルールの追加・編集は必ず `AGENTS.md` を直接編集してください。
---

# Repository Guidelines

> **Note**: `CLAUDE.md` は `AGENTS.md` へのシンボリックリンクです。ルールの追加・編集は必ず `AGENTS.md` を直接編集してください。

## 必須ルール

### Worktree必須
変更作業は、**必ず git worktree を作成してから開始すること**。メインのリポジトリディレクトリでは直接変更を行わない。

```bash
# 1. worktreeを作成（必ずdevelopから分岐すること）
git worktree add ../mirai-gikai-<branch-name> -b <branch-name> develop

# 2. settings.local.jsonをコピー（権限設定のため必須）
mkdir -p ../mirai-gikai-<branch-name>/.claude
cp .claude/settings.local.json ../mirai-gikai-<branch-name>/.claude/

# 3. .envをコピー（環境変数の引き継ぎ）
cp .env ../mirai-gikai-<branch-name>/

# 4. 依存パッケージをインストール
cd ../mirai-gikai-<branch-name> && pnpm install --frozen-lockfile
```

- **必ず `develop` から分岐する**: `git worktree add` の末尾に `develop` を指定すること。省略すると現在のブランチ（HEADが別ブランチを指している場合）から分岐し、無関係なコミットがPRに混入する原因になる。

- **目的**: developブランチを常にクリーンに保ち、作業の分離と並列作業を容易にする
- **developに変更が残っている場合のリカバリ**: worktreeを作成する前に、developブランチの変更を必ずクリーンアップすること。作業途中の変更をdevelopに残したままworktreeを作成・作業することは禁止。
  ```bash
  # 変更を退避してからworktreeを作成
  git stash --include-untracked
  git worktree add ../mirai-gikai-<branch-name> -b <branch-name>
  # worktreeに移動して退避した変更を適用
  cd ../mirai-gikai-<branch-name>
  git stash pop
  ```

### 実装完了後は即PR作成
実装完了後は「コミットしますか？」等の確認を挟まず、コミット → push → PR作成まで一気に進めること。ユーザーへの確認は不要。

### セルフレビュー必須
実装完了後（コミット前）に、以下の順で必ずセルフレビューを実施すること：

1. **`/simplify` を実行**: 変更コードの重複・可読性・効率の観点から自己修正を行う。明らかな問題を先に潰しておくことで、後段の `/review` の指摘ノイズを減らす。
2. **`/review` を実行**: Codexレビュー・`test-guidelines-checker` によるテストガイドラインチェック・`code-quality-checker` によるコード品質チェックを同時に実行する。指摘があれば修正する。

両方を通過したら、ユーザーに確認せずそのままコミット → push → PR作成まで一気に進めること（`gh pr create`）。

### UI変更時のスクリーンショット必須
PR作成後、変更差分にUI関連ファイル（`web/src/`, `admin/src/` 配下の `.tsx`, `.css` 等）が含まれる場合は、必ず `/pr-screenshot` スキルを実行すること。スキルが自動でdevサーバー起動→スクリーンショット撮影→R2アップロード→PR本文更新まで行う。

### 並列PR作成
複数の独立したPRを作成する場合は `/parallel-pr` スキルを使用すること。

### Linearタスクのステータス管理
Linear issue ID（例: `MIR-123`）を含むタスクを依頼された場合、以下のタイミングで `/linear` スキルを実行すること：
- **作業着手時**: `/linear start <issue-id>` でステータスを `In Progress` に更新
- **PR作成後**: `/linear review <issue-id>` でステータスを `In Review` に更新し、PR URLをissueにリンク

## Project Structure & Module Organization
- `web/` は公開用 Next.js アプリ。共通 UI は `src/components`、Vitest のテストは `src/**/*.test.ts` に配置します。
- `admin/` はポート 3001 で動く管理用 Next.js。審議フローやダッシュボードはここに集約します。
- `packages/supabase/` は共有 Supabase クライアントと型定義を提供し、生成結果は `types/` に保存します。
- `packages/shared/` は `web` と `admin` の両方で使う共通ロジック（AI モデル定義、ストリーム処理等）を提供します。
- `packages/seed/` はローカルデータ投入用の TypeScript スクリプト (`run.ts`, `data.ts`) を管理します。
- `supabase/` はマイグレーションと設定ファイルを保持します。
- 設計ドキュメントは `docs/` に格納し、ルートの設定ファイル（`biome.json`, `pnpm-workspace.yaml` など）は全体ポリシーとして扱います。
- **web と admin でのコード共有**: 同一ロジックを `web/` と `admin/` の両方で使う場合は、`packages/` 配下の workspace パッケージに切り出すこと。同じコードを両アプリに重複配置するのは禁止。既存の `@mirai-gikai/shared` パッケージに追加するか、用途に応じて新しいパッケージを作成する。

## Next.js アーキテクチャ指針
- Bulletproof React の feature ベース構成を採用します。
- `app/` 配下の `page.tsx` は、URL パラメータ（`params` や `searchParams`）の取得と Feature コンポーネントへの受け渡しのみを担当する薄いラッパーとし、ビューやロジックは `features/` 配下に実装します。
- export 用の `index.ts` は作成せず、必要なファイルから直接 import します。
- Server Components を標準とし、状態管理・イベント処理が必要な場合のみ `"use client"` を付与した Client Component を追加します。
- ファイル名はケバブケース、コンポーネントはパスカルケース、関数はキャメルケースで統一します。

### Feature ディレクトリ構造
複雑な feature では server/client/shared の3層構造を採用します：

```
src/features/{feature}/
├── server/
│   ├── repositories/  # データアクセス層（Supabase呼び出しを集約）
│   ├── components/    # Server Components
│   ├── loaders/       # Server Components用データ取得関数
│   ├── actions/       # Server Actions ("use server")
│   ├── services/      # ビジネスロジック層
│   └── utils/         # Server専用ユーティリティ
├── client/
│   ├── components/    # Client Components（Server/Client両方で使えるものも含む）
│   ├── hooks/         # カスタムフック
│   └── utils/         # Client専用ユーティリティ
└── shared/
    ├── types/         # 共通型定義
    └── utils/         # 共通ユーティリティ
```

`web/` と `admin/` の両方で同じ server/client/shared 構成を採用します。ただし `admin/` では Server Components が中心のため `client/` ディレクトリを省略している feature もあります。

- Server側ファイルには `"server-only"` を、Client Componentsには `"use client"` を付与
- 型定義やServer/Client両方で使う関数は `shared/` に配置
- **純粋関数の切り出し**: 新規実装時、外部依存（DB・API・認証等）を持たない計算・変換・判定ロジックは純粋関数として `utils/` に切り出すこと。配置先は用途に応じて `shared/utils/`、`server/utils/`、`client/utils/` を選択する。
- シンプルな feature は従来の `components|actions|api|types` 構成でも可

Repository レイヤーの詳細は [docs/repository-layer.md](docs/repository-layer.md) を参照。

## Build, Test, and Development Commands
- 依存導入は `pnpm install`、全てのスクリプトは pnpm 経由で実行します。
- `pnpm dev` は `.env` を共有しつつ `web`・`admin`・各パッケージの dev サーバーを並列起動します。
- `pnpm test` でワークスペース横断の Vitest を実行。局所実行は `pnpm --filter web test` や `test:watch` を利用します。
- 品質ゲートとして `pnpm lint`（Biome format+lint）と `pnpm typecheck` を PR 前に通過させます。
- DB 関連は `pnpm db:reset`、`pnpm db:migrate`、`pnpm db:types:gen`、`pnpm seed` を用途に応じて組み合わせます。

## Coding Style & Naming Conventions
- Biome が 2 スペースインデント、LF、ダブルクォート、セミコロン、80 文字幅を強制します。
- React コンポーネントと公開型は PascalCase、フックやユーティリティは camelCase を維持します。
- ファイル名は `bill-contents-data.ts` のようにローワーハイフンで表記し、スタイルは Tailwind ユーティリティを先に検討します。
- **アイコン**: インラインSVGは禁止です。必ず `lucide-react` からアイコンコンポーネントをインポートして使用してください。
- **ボタン**: `<button>` タグの使用は禁止です。必ず `@/components/ui/button` の `Button` コンポーネントを使用してください。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [team-mirai/mirai-gikai](https://github.com/team-mirai/mirai-gikai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
