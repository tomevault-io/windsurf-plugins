---
trigger: always_on
description: このファイルは、Claude Code (claude.ai/code) がこのリポジトリで作業する際のガイドを提供します。
---

# CLAUDE.md

このファイルは、Claude Code (claude.ai/code) がこのリポジトリで作業する際のガイドを提供します。

## プロジェクト概要

file-funeralは、ローカルストレージとクラウドプロバイダー間でファイルをバックアップ・同期するクラウドネイティブなデスクトップアプリケーションです。主な用途は、複数PC間での双方向ファイル同期によるファイル共有です。

**主要機能（計画）:**
- ローカルファイルをクラウドストレージ（AWS S3、GCS、S3互換）にアップロード/バックアップ
- クラウドからローカルへのファイルダウンロード
- ローカルとクラウド間の双方向同期
- 同期状態インジケーター付きファイルリストの視覚的表示
- 「両方保存」方式による競合解決（Dropboxスタイル）

**詳細な要件**: `REQUIREMENTS.md` を参照してください。

---

## 技術スタック

### バックエンド
- **フレームワーク**: Tauri v2.9.5
- **言語**: Rust (edition 2021, 1.77.2以降)
- **非同期ランタイム**: Tokio（計画中）
- **クラウドSDK**:
  - v1.0 (計画): `aws-sdk-s3`
  - v2.0以降 (計画): S3互換、GCS SDK
- **プラグイン**:
  - `tauri-plugin-log` (インストール済み)
  - `tauri-plugin-fs` (計画中、ファイルシステムアクセス)
- **認証情報ストレージ**: `keyring` クレート（計画中、OSキーチェーン）

### フロントエンド
- **ビルドツール**: Vite 7.3.0
- **フレームワーク**: React 19.2.3
- **言語**: TypeScript 5.9.3
- **開発サーバー**: `http://localhost:1420`
- **ビルド出力**: `dist/`ディレクトリ

### 現在インストール済みの依存関係

**Rust (src-tauri/Cargo.toml):**
```toml
[dependencies]
serde_json = "1.0"
serde = { version = "1.0", features = ["derive"] }
log = "0.4"
tauri = { version = "2.9.5", features = [] }
tauri-plugin-log = "2"
```

**Node.js (package.json):**
- React 19.2.3 + React DOM
- TypeScript 5.9.3
- Vite 7.3.0 + @vitejs/plugin-react
- ESLint + TypeScript ESLint
- @tauri-apps/cli 2.9.6

### 計画中の依存関係（v1.0 MVP）

```toml
# 以下は実装フェーズで追加予定
aws-config = { version = "1.1.7", features = ["behavior-version-latest"] }
aws-sdk-s3 = "1.117.0"
tokio = { version = "1", features = ["full"] }
tauri-plugin-fs = "*"
keyring = "*"
anyhow = "*"
thiserror = "*"
chrono = "*"
```

---

## プロジェクト構造

### 現在の構造

```
file-funeral/
├── src-tauri/              # Rustバックエンド
│   ├── src/
│   │   ├── main.rs         # エントリポイント（app_lib::run()を呼び出し）
│   │   └── lib.rs          # メインロジック + ユニットテスト
│   ├── tests/
│   │   └── integration_test.rs  # 統合テスト
│   ├── Cargo.toml          # Rust依存関係
│   ├── tauri.conf.json     # Tauri設定
│   └── build.rs            # ビルドスクリプト
├── src/                    # Reactフロントエンド
│   ├── main.tsx            # Reactエントリポイント
│   ├── App.tsx             # メインコンポーネント
│   └── App.css
├── dist/                   # Viteビルド出力（.gitignore）
├── index.html              # HTMLテンプレート
├── vite.config.ts          # Vite設定
├── tsconfig.json           # TypeScript設定
├── eslint.config.js        # ESLint設定
├── REQUIREMENTS.md         # 詳細要件定義書
├── TODO.md                 # 実装タスク一覧
└── CLAUDE.md              # このファイル
```

### 計画中の構造（v1.0 MVP実装後）

```
src-tauri/src/
├── main.rs                 # エントリポイント
├── lib.rs                  # 公開API + setup
├── commands.rs             # Tauri Commands（フロントエンド ↔ バックエンド通信）
├── storage/                # クラウドストレージ抽象化層
│   ├── mod.rs              # CloudStorageProvider trait定義
│   ├── s3.rs               # AWS S3実装
│   └── types.rs            # FileInfo, FileMetadata等
├── sync/                   # 同期エンジン
│   ├── mod.rs              # 同期ロジック
│   ├── conflict.rs         # 競合解決
│   └── metadata.rs         # メタデータ管理
└── auth/                   # 認証情報管理
    └── mod.rs              # Keyring統合
```

---

## Tauri固有の開発ノート

### アーキテクチャの理解

**main.rs vs lib.rs:**
- `main.rs`: 最小限のエントリポイント。`app_lib::run()`を呼び出すのみ
- `lib.rs`: 実際のアプリケーションロジック、Tauri setupコード、ユニットテストを含む
- この構成により、ライブラリとしてもバイナリとしてもビルド可能

**Tauri Commands:**
- `#[tauri::command]`マクロでRust関数をフロントエンドに公開
- フロントエンドから`invoke('command_name', { args })`で呼び出し
- 非同期処理はasync fnで実装可能
- エラーは`Result<T, E>`で返し、フロントエンドでキャッチ

**テスト構造:**
- `lib.rs`内の`#[cfg(test)] mod tests`: ユニットテスト
- `tests/`ディレクトリ: 統合テスト（外部クレートとしてアプリをテスト）
- `npm test`で両方を実行（`cd src-tauri && cargo test`）

**設定ファイル:**
- `tauri.conf.json`: アプリケーション設定
  - `devUrl`: `http://localhost:1420`（Vite開発サーバー）
  - `frontendDist`: `../dist`（本番ビルド時の静的ファイル）
  - `beforeDevCommand`: `npm run dev`（開発時にViteを起動）
  - `beforeBuildCommand`: `npm run build`（ビルド時にフロントエンドをビルド）

---

## 計画中のアーキテクチャ（v1.0 MVP）

### ストレージ抽象化レイヤー

アプリはクラウドストレージプロバイダーのためのトレイトベースの抽象化を使用します:

```rust
// src-tauri/src/storage/mod.rs (計画中)
pub trait CloudStorageProvider {
    async fn upload(&self, path: &str, data: &[u8], metadata: FileMetadata) -> Result<()>;
    async fn download(&self, path: &str) -> Result<Vec<u8>>;
    async fn list(&self, prefix: &str) -> Result<Vec<FileInfo>>;
    async fn delete(&self, path: &str) -> Result<()>;
    async fn get_metadata(&self, path: &str) -> Result<FileMetadata>;
}

pub struct FileInfo {
    pub path: String,
    pub size: u64,
    pub last_modified: DateTime<Utc>,
    pub etag: Option<String>,
}

pub struct FileMetadata {
    pub size: u64,
    pub last_modified: DateTime<Utc>,
    pub content_type: Option<String>,
    pub etag: Option<String>,
}
```

**実装計画:**
- v1.0: `S3Provider` (AWS S3)
- v2.0: `S3CompatibleProvider` (MinIO、Backblaze B2等)
- v2.5: `GCSProvider` (Google Cloud Storage)

### 同期エンジン

**同期タイミング:**
- アプリ起動時（自動）
- ユーザーが「同期」ボタンをクリックしたとき（手動）
- （オプション）アプリ終了時

**競合解決:**
- 競合検知時、両方のファイルを保存
- クラウド版: `filename.txt`
- ローカル版: `filename (PC名's conflicted copy YYYY-MM-DD).txt`

**削除の扱い:**
- v1.0: 削除を同期（完全同期）
- v2.0以降: ソフト削除（ゴミ箱/復元機能付き）

---

## 開発コマンド

### セットアップ
```bash
# リポジトリクローン後、最初に実行
npm install

# Tauri CLIをグローバルにインストール（推奨）
cargo install tauri-cli
```

### 開発
```bash
# 開発モードで実行（フロントエンド + バックエンドを同時起動）
npm run tauri:dev

# フロントエンドのみ開発サーバー起動（Tauri無し）
npm run dev

# 本番ビルド（デスクトップアプリケーションとしてパッケージ化）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ohr486) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
