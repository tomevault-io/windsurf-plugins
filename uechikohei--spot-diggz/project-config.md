---
trigger: always_on
description: spot-diggz: スケートスポット検索・シェアアプリ（旧SkateSpotSearchのリプレイス）
---

# CLAUDE.md

必ず日本語で回答すること。

## プロジェクト概要

spot-diggz: スケートスポット検索・シェアアプリ（旧SkateSpotSearchのリプレイス）

- バックエンド: Rust（スクラッチ実装、フレームワーク非使用） → `web/api/`
- フロントエンド: React + TypeScript → `web/ui/`
- インフラ: GCP (Cloud Run, Firestore, BigQuery, Cloud Functions) + Terraform → `web/resources/`
- モバイル: iOS (`iOS/`), Android (`android/`) ※予定
- CI/CD: GitHub Actions (`.github/workflows/ci.yml`)
- リポジトリ: `uechikohei/spot-diggz`（モノレポ構成）

## ビルド・テスト・Lintコマンド

### Rust API (`web/api/`)

```bash
cargo fmt -- --check          # フォーマットチェック
cargo clippy -- -D warnings   # Lintチェック
cargo test --verbose          # ユニットテスト
cargo build --release         # リリースビルド
cargo run                     # ローカル起動 (localhost:8080)
```

### React UI (`web/ui/`)

```bash
npm run lint                  # ESLint
npm run type-check            # TypeScript型チェック
npm test -- --coverage --watch=false  # ユニットテスト
npm run build                 # プロダクションビルド
npm run dev                   # ローカル起動 (localhost:3000)
```

### Terraform (`web/resources/`)

```bash
terraform fmt -check -recursive   # フォーマットチェック
terraform init -backend=false && terraform validate  # バリデーション
```

## コミット前の必須チェック

**pre-commitフック（Husky）が以下を自動実行する:**

- Prettier による自動フォーマット（JS/TS/JSON/CSS/MD）
- ESLint チェック（TS/TSX変更時）
- `cargo fmt --check` + `cargo clippy`（.rs変更時）
- `terraform fmt -check`（.tf変更時）

**テストはpre-commitに含まれない**ため、`/verify` で別途実行すること:

1. Rust: `cd web/api && cargo test`
2. React: `cd web/ui && npm run type-check && npm test -- --watch=false`

## コーディング規約

### 命名規約（4層ティア）

プロジェクト全体で以下の4層命名規約を適用する。

| Tier | 名称           | 形式                  | 用途                                              | 例                         |
| ---- | -------------- | --------------------- | ------------------------------------------------- | -------------------------- |
| 1    | Display Name   | `SpotDiggz`           | UI表示、ロゴ、ドキュメント見出し                  | `<title>SpotDiggz</title>` |
| 2    | Machine Name   | `spotdiggz`           | リポジトリ名、ディレクトリ名、ドメイン            | `uechikohei/spotdiggz`     |
| 3    | Infra Resource | `sdz-{env}-{purpose}` | GCPリソース名（全小文字・ハイフンのみ）           | `sdz-dev-img-spots`        |
| 4    | Source Code    | 言語規約準拠          | コード内識別子（`sdz`/`Sdz`/`SDZ`プレフィックス） | `sdzUserProfile`           |

#### Tier 1: Display Name（表示名）

- **`SpotDiggz`**（PascalCase、スペースなし）
- UI上のアプリ名、ロゴ、ドキュメント見出し、OGP等で使用

#### Tier 2: Machine Name（マシン名）

- **`spotdiggz`**（全小文字、区切りなし）
- リポジトリ名、ディレクトリ名、ドメイン、パッケージ名等で使用

#### Tier 3: Infrastructure Resource Name（インフラリソース名）

- **全小文字・ハイフンのみ**（アンダースコア禁止）
  - GCP/AWSタグの小文字正規化でキー衝突を防止
  - BigQuery/Athena連携時のアンダースコア制約を回避
- セグメント構成: `sdz-{env}-{purpose}`
- 冗長な型サフィックスは付けない（例: Cloud Storageに`bucket`は不要）

| リソース          | 命名パターン            | 例                    |
| ----------------- | ----------------------- | --------------------- |
| GCPプロジェクト   | `sdz-{env}`             | `sdz-dev`             |
| Cloud Run         | `sdz-{env}-api`         | `sdz-dev-api`         |
| 画像Storage       | `sdz-{env}-img-spots`   | `sdz-dev-img-spots`   |
| UIホスティング    | `sdz-{env}-ui-hosts`    | `sdz-dev-ui-hosts`    |
| Service Account   | `sdz-{env}-{role}-sa`   | `sdz-dev-api-sa`      |
| Artifact Registry | `sdz-{env}-api`         | `sdz-dev-api`         |
| WIF Pool          | `sdz-{env}-github-pool` | `sdz-dev-github-pool` |

#### Tier 4: Source Code Name（ソースコード名）

全識別子に `sdz`/`Sdz`/`SDZ` プレフィックスを付与する。

| 言語       | 変数・関数                       | 型・構造体                  | 定数                                  | ファイル名            |
| ---------- | -------------------------------- | --------------------------- | ------------------------------------- | --------------------- |
| TypeScript | `camelCase`: `sdzUserProfile`    | `PascalCase`: `SdzSpotData` | `SCREAMING_SNAKE`: `SDZ_API_BASE_URL` | `SdzSpotCard.tsx`     |
| Rust       | `snake_case`: `sdz_user_profile` | `PascalCase`: `SdzSpotData` | `SCREAMING_SNAKE`: `SDZ_MAX_SPOTS`    | `sdz_spot_service.rs` |
| Terraform  | `snake_case`: `sdz_api`          | -                           | -                                     | `sdz_cloud_run.tf`    |

### Gitコミットメッセージ

- プレフィックスは英語（Conventional Commits準拠）: `feat` / `fix` / `docs` / `refactor` / `test` / `chore` / `style` / `perf` / `ci`
- スコープ: `(ios)` / `(api)` / `(web)` / `(infra)` / `(repo)`
- タイトル・詳細メッセージは日本語で記載
- 詳細（-m body）は箇条書きで変更内容を簡潔に記載
- Co-Authored-By行は付けない（著者はuechikoheiのみとする）

```
feat(ios): スポット投稿画面のバリデーションを追加

- 必須項目の未入力チェックを実装
- エラーメッセージを日本語で表示
```

### ブランチ戦略

GitHub Flow を使用する

## 課題管理（GitHub Issues/Projects）

### 起票フォーマット（自動判定）

`/issue` コマンドで起票時、内容に応じてフォーマットを自動選択する:

| 判定条件                           | フォーマット                                 | ラベル                |
| ---------------------------------- | -------------------------------------------- | --------------------- |
| 既に発生した障害・バグ・技術調査   | **4F形式**（Fact/Find/Fix/Future）           | `troubleshooting`     |
| 未来の検討・新機能・設計・要件整理 | **STAR形式**（Situation/Task/Action/Result） | `planning` / `design` |
| 上記に当てはまらないナレッジ・メモ | **フリーフォーマット**                       | `knowledge`           |

### 起票時の必須設定

- label: `troubleshooting` / `planning` / `design` / `knowledge`
- Priority: `P0`（本番障害）/ `P1`（開発に支障）/ `P2`（改善・バックログ）

### トラブルシューティング自動起票

Claude Codeの作業中にトラブルシューティングが発生し解消した場合、自動的に `/issue` を実行して4F形式で知見を記録する（起票前にユーザー承認を取る）。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/uechikohei) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
