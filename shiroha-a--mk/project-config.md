---
trigger: always_on
description: このファイルは、このリポジトリで作業する際にClaude Codeが参照するプロジェクト固有のガイドラインです。
---

# CLAUDE.md

このファイルは、このリポジトリで作業する際にClaude Codeが参照するプロジェクト固有のガイドラインです。

本プロジェクトはMisskey（TypeScript/NestJS製の分散型SNS）をGoで書き換えるリライトプロジェクトです。オリジナルMisskeyとのAPI互換性・ActivityPub連合互換性の維持を最優先とします。

タスク管理はGitHub Issues / Pull Requestsで行います（詳細はSection 7）。

## 1. 技術スタック

### コア

| Component | Library | 用途 |
|-----------|---------|------|
| 言語 | **Go 1.26** | `go.mod`でバージョン管理 |
| Webフレームワーク | **Echo v4** (`labstack/echo/v4`) | HTTPルーティング、ミドルウェア、WebSocket |
| ORM | **GORM** (`gorm.io/gorm`) | PostgreSQLアクセス |
| Migration | **golang-migrate** (`golang-migrate/migrate/v4`) | SQLベースのマイグレーション |
| Config | **Viper** (`spf13/viper`) | YAML + 環境変数オーバーライド |
| Logging | **slog** (標準ライブラリ) | 構造化ロギング |

### インフラ

| Component | Library | 用途 |
|-----------|---------|------|
| PostgreSQL Driver | **pgx/v5** (`jackc/pgx/v5`) | PostgreSQL接続 |
| Redis | **go-redis v9** (`redis/go-redis/v9`) | キャッシュ、PubSub |
| Job Queue | **mkq** (`shiroha-a/mkq`) | BullMQ wire互換のRedisジョブキュー（既定）。`asynq` (`hibiken/asynq`) はlegacyで削除予定 |
| Search | **meilisearch-go** | Meilisearch連携 |
| Object Storage | **aws-sdk-go-v2/s3** | S3互換ストレージ |

### 連合 / ActivityPub

- **HTTP Signatures**: 自前実装（`internal/activitypub/`）
- **JSON-LD**: `piprate/json-gold` (LD-Signature の canonicalize。`internal/activitypub/ld/`)
- **ActivityStreams Types**: カスタム構造体

### 認証

- **bcrypt** (`golang.org/x/crypto/bcrypt`) - パスワードハッシュ
- **pquerna/otp** - TOTP（2FA）
- **go-webauthn/webauthn** - パスキー / セキュリティキー（2FA、`signin-with-passkey`）

### テスト

- **testing** (標準) + **testify** (`stretchr/testify`)
- **testcontainers-go** - 実PostgreSQL/Redisを使った統合テスト
- 単体テストでは`internal/testutil/`のモックを使用

## 2. Project Structure

```
/
├── cmd/
│   ├── misskey/            # メインバイナリのエントリポイント
│   ├── migrate/            # マイグレーションCLIツール
│   ├── backfill-note-tags/ # note.tags を NFKC 正規化し直す一回限りのバッチ
│   ├── backfill-remote-host/ # 保存済みリモート host を punycode 正規化し直すバッチ
│   └── dbgtimeline/        # home/global timeline の JSON encoder panic を再現するデバッグ用ツール
├── internal/               # 全23パッケージ
│   ├── config/             # 設定ローダー（Misskey YAML互換）
│   ├── db/                 # GORM の PostgreSQL 接続配線
│   ├── server/             # HTTPサーバーのセットアップ、ルーティング、ミドルウェア
│   ├── api/                # APIハンドラ（エンドポイント単位でサブディレクトリ）
│   │   ├── admin/          # admin/* 管理API
│   │   ├── ap/             # ap/* ActivityPub解決API
│   │   ├── auth/           # auth/* 認証API
│   │   ├── notes/          # notes/* ノート関連API
│   │   ├── users/          # users/* ユーザー関連API
│   │   ├── i/              # i/* 自アカウントAPI
│   │   ├── drive/          # drive/* ファイル管理API
│   │   ├── federation/     # federation/* 連合情報API
│   │   └── ...             # その他エンドポイント群
│   ├── core/               # ビジネスロジック層（サービス）
│   ├── activitypub/        # ActivityPub実装（Inbox、Deliver、Renderer、Resolver、HTTP署名、LD-Signature）
│   ├── model/              # DBモデル（GORM、Misskeyエンティティ対応）
│   ├── repository/         # データアクセス層
│   ├── queue/              # ジョブキュー（既定mkq / legacy asynq）とプロセッサ
│   ├── stream/             # WebSocketストリーミング（チャンネル実装）
│   ├── entity/             # レスポンス用DTO（シリアライゼーション）
│   ├── entitycompat/       # 静的な shape drift 検出と doc gate（Section 8 / docs/shape-drift.md）
│   ├── pluginspec/         # 公開プラグインAPIの面を抽出（entitycompat が使う）
│   ├── pluginstore/        # プラグインごとの専用 PostgreSQL schema (#2481)
│   ├── safehttp/           # 外向きHTTPの共通ヘルパー（SSRFガード等）
│   ├── charttick/          # チャートの絶対時刻を再導出する TickFunc 群
│   ├── maintenance/        # SQL migration として書けない後始末バッチ（`cmd/` の CLI から手動で回す）
│   ├── frontendutil/       # 同梱フロントエンドの資産配信ヘルパー
│   ├── pgarray/            # database/sql 用の PostgreSQL 配列型
│   ├── sentry/             # sentry-go の配線
│   ├── redislog/           # go-redis の内部ロガーを slog へ流す配線
│   ├── misc/               # ユーティリティ（ID生成 等。既定は`aidx`、Section 6 参照）
│   └── testutil/           # テスト用ヘルパー（testcontainers、モック）
├── plugin/                 # プラグインが import する公開パッケージ（docs/plugins/）
├── plugins/                # プラグイン本体。gitignore 済で同梱するものだけ例外指定
├── tools/                  # parity ゲート / コード生成のCLI群（apicompat、shapediff、pluginbuild 等）
├── migration/              # golang-migrate用SQLファイル（`NNNNNN_name.up.sql` / `.down.sql`）
├── test/                   # Go の e2e（`test/e2e` / `test/e2e_federation`）
├── tests/                  # Go 以外の検証基盤（playwright / diff / dropin / bench / upstream-e2e 等）
├── third_party/misskey/    # fork した Misskey TS（submodule。フロントエンドの供給元）
├── deploy/                 # デプロイ用の補助資材（UDS 構成、pg_bigm 入り postgres image）
├── .config/                # 設定ファイル（Misskey互換YAML）
│   ├── default.yml.example # ローカル開発用テンプレート (track 対象)
│   ├── docker.yml.example  # Docker Compose用テンプレート (track 対象)
│   ├── default.yml         # operator-local (gitignored)
│   └── docker.yml          # operator-local (gitignored)
├── docs/                   # プロジェクトドキュメント
├── Makefile
├── Dockerfile
├── docker-compose.yml      # **`name:` が無い**。単体で使うと本番 project `mk` に合流する
└── go.mod                  # Moduleパス: github.com/shiroha-a/mk
```

`built/` と `drive-files/` は gitignored な生成物 / ローカルストレージ。

レイヤ責務：
- **api** → **core** → **repository** → **model** の順に依存。逆向きの依存は禁止。
- **entity**はレスポンス変換専用。ドメインロジックを入れない。
- **activitypub**は`core`から呼び出され、連合処理を担う。

## 3. Development Commands

すべて`Makefile`経由で実行できます。

```bash
# ビルド
make build                  # ./built/misskey に実行ファイル生成

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shiroha-a/mk](https://github.com/shiroha-a/mk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
