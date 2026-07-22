---
trigger: always_on
description: - yarn や poetry はそれぞれ `client/` と `server/` のディレクトリに移動した状態で実行してください。ルートディレクトリにはパッケージ管理系のファイルは一切配置していません。
---

# AGENTS.md

## プロジェクト固有の注意事項

- yarn や poetry はそれぞれ `client/` と `server/` のディレクトリに移動した状態で実行してください。ルートディレクトリにはパッケージ管理系のファイルは一切配置していません。
- サーバー側では poetry を使っているので、python コマンドは必ず全て poetry run 経由で実行します。python を直接実行すると .venv/ 以下のライブラリがインストールされていないために失敗します。

## 開発環境構成

### サーバー API (port 7000、常にユーザー管理)

- 依頼を受けた時点で、サーバー API は次のいずれかの状態で常駐しています。**いずれもユーザーが管理しているプロセスであり、エージェントが直接起動・停止すべきではありません**
  - **リロードモード**: `server/` で `poetry run task dev` で起動。コード変更が hot reload されます。基本的にこの状態で依頼が来ます
  - **リロードなしの開発サーバー**: `server/` で `poetry run task serve` で起動。hot reload なしのユーザー権限プロセスです
  - **pm2 常駐**: `sudo pm2 start KonomiTV` で起動。KonomiTV は root 側の pm2 プロファイルにしかインストールされていないため、`pm2` コマンドの実行には必ず `sudo` が必要であり、**エージェントがユーザーの許可なく `pm2` を実行することはできません**
- FastAPI の listen ポートは常に 7000 で固定です (Akebi HTTPS Server が `127.0.0.77:7010` をリバースプロキシしています)
- サーバー側コードを変更して挙動を確認したい場合の手順:
  - リロードモードで動いている場合は、変更が自動で反映されます
  - リロードなし開発サーバー / pm2 常駐で動いている場合は、**ユーザーに「リロードモードでの起動への切り替え、もしくはサーバー再起動」を依頼してください**
- エージェントが直接 `python KonomiTV.py` や `poetry run python KonomiTV.py` を実行するのは禁止です。サーバーの起動には必ず taskipy で定義済みの `poetry run task serve` / `poetry run task dev` を使用してください (それでも、上記の通り既存プロセスとの衝突を避けるためエージェント自身が起動することは原則避けてください)

### クライアント開発サーバー (port 7001、必要ならエージェントが起動可)

- クライアントの開発サーバーは普段は起動していません
- UI を検証する必要がある場合は、`client/` で `yarn dev` をエージェントが起動して構いません
  - 起動すると port 7001 で Akebi HTTPS Server 経由でリッスンされます (内部の Vite は `127.0.0.77:7011` でリッスンします)
- **重複起動は禁止**です。起動前に必ず `ps -ef | grep vite` などで既存プロセスの有無を確認してください
- `yarn dev` で起動するクライアントは、開発モード時のみ同じドメインの `:7000` のサーバー API を直接叩くようハードコードされています ([client/src/utils/Utils.ts](client/src/utils/Utils.ts) の `Utils.api_base_url` を参照)。Vite の proxy 設定は不要です
- Chrome DevTools MCP からの検証時は `https://my.local.konomi.tv:7001` にアクセスしてください
- クライアント開発サーバー経由で API リクエストが想定通りに動かない場合でも、**サーバーを立て直そうとしないでください**。まず `Utils.api_base_url` の DEV 分岐の挙動を読み直し、port 7000 で動いているサーバー側の状態を `ps -ef | grep KonomiTV` などで確認してください

### Docker 版ステージング (port 7100、別物)

- `/Develop/KonomiTV-Docker` 以下には別途 Docker 版のステージング環境があります (port 7100、内部 HTTP は `127.0.0.77:7110`)
- 本リポジトリの開発環境とは独立した別プロセスです

### HTTPS が必須な理由

- KonomiTV はクリップボードなど Secure Context (HTTPS) でしか動作しない API を使用しています
- localhost 以外でも正規の HTTPS で提供できるよう、Akebi HTTPS Server が `akebi.konomi.tv` の keyless server を経由してリバースプロキシを行っています
- HTTP に直接アクセスされると Secure Context API が動かず混乱を招くため、内部の HTTP は `127.0.0.77` でリッスンする構成になっています

## 技術スタック

KonomiTV は、クライアント・サーバーアーキテクチャに基づく Web アプリケーション (PWA) です。
以下の2つの主要部分で構成されています。

KonomiTV が一般的な Web サービスと異なる点は、フロントエンドと API サーバーの両方が各ユーザーの PC 環境で動作する点です。
したがって、Windows と Linux の両方で動作するように開発する必要があります。
Windows では Windows サービス、Linux では pm2 サービスとして動作するよう設計しています。

- `client/`: KonomiTV のフロントエンドアプリケーション (PWA)
  - TypeScript
  - yarn v1
  - Vite
  - Vue.js 3.x
    - Vuetify 3.x
    - Pinia
- `server/`: KonomiTV のバックエンド API サーバー
  - Python 3.11
  - Poetry
  - Uvicorn
  - FastAPI
    - Pydantic v2
  - Tortoise ORM
    - SQLite (ローカル動作が必要なため MySQL や PostgreSQL は採用できなかった)
    - Aerich

## ディレクトリ構成

### クライアント (`client/`)

- `public/`: 直接提供される静的ファイル
- `src/`: ソースコード
  - `views/`: Vue ルートコンポーネント/ページ
    - `TV/`: テレビ視聴関連ページ
    - `Videos/`: 動画関連ページ
    - `Reservations/`: 予約関連ページ
    - `Settings/`: アプリケーション設定ページ
    - `Login.vue`: ログインページ
    - `Register.vue`: アカウント登録ページ
    - `MyList.vue`: マイリストページ
    - `WatchedHistory.vue`: 視聴履歴ページ
    - `MyPage.vue`: マイページ
    - `NotFound.vue`: 404 エラーページ
  - `components/`: Vue コンポーネント
    - `Watch/`: テレビ・録画番組視聴画面向けコンポーネント群
      - `Panel/`: 視聴画面右側のパネル内表示用コンポーネント群
        - `Twitter/`: ツイート検索/タイムライン表示/キャプチャ管理/ツイート表示用コンポーネント群
    - `Settings/`: 設定ページから呼び出されるダイアログコンポーネント群
    - `HeaderBar.vue`: ヘッダーバー
    - `SPHeaderBar.vue`: スマートフォン用ヘッダーバー
    - `Navigation.vue`: ナビゲーション
    - `BottomNavigation.vue`: スマートフォン用下部ナビゲーション
    - `Snackbars.vue`: 通知メッセージ表示コンポーネント
    - `Breadcrumbs.vue`: パンくずリスト表示コンポーネント
  - `stores/`: 状態管理 (Pinia ストア)
  - `services/`: サーバー API へのサービスクライアント
    - `player/`: KonomiTV の視聴画面で用いられるライブ/ビデオプレイヤーのロジック (重要)
      - `managers/`: PlayerController に紐づく様々な機能のロジックを提供し、各機能に責任を持つ PlayerManager 群
      - `PlayerController.ts`: 動画プレイヤーである DPlayer に関連するロジックを丸ごとラップするクラスで、KonomiTV の再生系ロジックの中核を担う
  - `utils/`: ユーティリティ関数とヘルパー
  - `workers/`: 重い処理をバックグラウンドで実行するための Web Workers コード (with Comlink)
  - `styles/`: グローバル CSS の定義 (グローバル CSS は `App.vue` の方がメイン)
  - `router/`: Vue Router 設定
  - `plugins/`: Vue プラグインの初期化定義
  - `App.vue`: アプリケーションのルートコンポーネント (グローバル CSS 定義もここに含まれる)
  - `main.ts`: アプリケーションのエントリーポイント・初期化処理
- `package.json`: Node.js プロジェクト設定と依存関係 (yarn)
- `vite.config.mts`: Vite ビルド設定
- `tsconfig.json`: TypeScript 設定
- `.eslintrc.json`: ESLint コードスタイル設定

### サーバー (`server/`)

- `app/`: FastAPI アプリケーションコード
  - `routers/`: API ルートハンドラー
    - `ChannelsRouter.py`: チャンネル関連メタデータ取得 API
    - `ProgramsRouter.py`: 番組関連メタデータ取得 API
    - `VideosRouter.py`: 録画番組メタデータ取得 API
    - `SeriesRouter.py`: 番組シリーズ関連 API
    - `LiveStreamsRouter.py`: 放送中テレビ放送のライブストリーミング配信関連 API
    - `VideoStreamsRouter.py`: 録画番組のストリーミング配信関連 API
    - `ReservationsRouter.py`: EDCB と連携したテレビ番組の録画予約関連 API
    - `ReservationConditionsRouter.py`: EDCB と連携したテレビ番組の自動録画予約条件 (EPG 自動予約) 関連 API
    - `DataBroadcastingRouter.py`: データ放送のインターネット接続機能向け API
    - `CapturesRouter.py`: キャプチャ画像管理 API
    - `TwitterRouter.py`: Twitter 連携 API

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tsukumijima/KonomiTV](https://github.com/tsukumijima/KonomiTV) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
