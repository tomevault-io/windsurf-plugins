---
trigger: always_on
description: このファイルは Claude Code がこのリポジトリで作業するための指示書です。
---

# CLAUDE.md

このファイルは Claude Code がこのリポジトリで作業するための指示書です。

## プロジェクト概要

**Ratatoskr** は個人用のセルフホスト型 RSS リーダーです。Cloudflare Workers 上で動作し、PC とスマートフォンの両方から利用します。

設計上の目標はただ一つ、**livedoor Reader / Fastladder の「流れるように読める」操作感を Web で再現すること**です。機能の豊富さや見た目の派手さではなく、**キーを押してから次の記事が出るまでの遅延がゼロであること**を最優先します。この判断基準は全ての実装判断に優先します。

詳細な設計根拠は `docs/DESIGN.md`、操作系の仕様は `docs/UX.md`、API 定義は `docs/API.md`、実装順は `docs/ROADMAP.md` を参照してください。**作業前に必ず該当するドキュメントを読んでください。**

## 技術スタック

| 領域 | 採用技術 |
| --- | --- |
| ランタイム | Cloudflare Workers (Static Assets 同梱) |
| DB | Cloudflare D1 (SQLite) |
| 定期実行 | Cron Triggers |
| フロントエンド | Vue 3 (Composition API) + TypeScript |
| ビルド | Vite |
| 状態管理 | Pinia |
| スタイル | Tailwind CSS v4 |
| クライアント永続化 | IndexedDB (`idb`) |
| XML パース | `fast-xml-parser` |
| HTML サニタイズ | Workers 組み込みの `HTMLRewriter` |
| 認証 | Cloudflare Access (Zero Trust) |
| テスト | Vitest + `@cloudflare/vitest-pool-workers`、E2E は Playwright |
| パッケージマネージャ | pnpm |

## ディレクトリ構成

```
.
├── CLAUDE.md
├── docs/               # 設計ドキュメント(この下は人間が管理。勝手に書き換えない)
├── migrations/         # D1 マイグレーション。連番 + 説明的な名前
├── src/                # Worker 側
│   ├── index.ts        # エントリポイント。fetch と scheduled の両ハンドラ
│   ├── retention.ts    # 保持期間による記事削除（1 日 1 回の cron）
│   ├── api/            # API ハンドラ。1 エンドポイント 1 ファイル
│   ├── crawler/        # フィード取得・パース・サニタイズ・保存
│   ├── db/             # D1 クエリ。SQL はここに閉じ込める
│   └── lib/            # 認証、エラー、共通型
├── web/                # Vue アプリ
│   ├── src/
│   │   ├── components/
│   │   ├── stores/     # Pinia
│   │   ├── lib/        # api クライアント、IndexedDB、keymap、prefetch
│   │   └── main.ts
│   └── index.html
├── shared/             # Worker と web で共有する型定義
└── wrangler.jsonc
```

## 開発コマンド

```bash
pnpm install

pnpm dev                # wrangler dev。Vite の HMR 込みでローカル起動
pnpm build              # web をビルドして Worker にバンドル
pnpm deploy             # 本番デプロイ

pnpm db:migrate         # ローカル D1 にマイグレーション適用
pnpm db:migrate:remote  # 本番 D1 に適用
pnpm db:console         # ローカル D1 に SQL を投げる

pnpm test               # Vitest (Worker + web のユニット)
pnpm test:e2e           # Playwright
pnpm typecheck          # vue-tsc + tsc --noEmit
pnpm lint               # ESLint + Prettier
```

`pnpm typecheck` と `pnpm test` は**コミット前に必ず通す**こと。

## 開発環境の注意点

M0 の実装で判明した、環境まわりの制約と手順。

### 初回セットアップ（クローン直後）

```bash
pnpm install       # esbuild / workerd の postinstall 許可は pnpm-workspace.yaml にコミット済み
pnpm db:migrate    # .wrangler/ は未コミット。クローン直後のローカル D1 は空
cp .dev.vars.example .dev.vars   # Access の secret を埋める（後述）
cp .env.example .env             # デプロイ後は Workers AI の接続にも要る（後述）
pnpm build                       # web/dist が無いと wrangler dev が起動しない（後述）
pnpm dev
```

`.dev.vars` は git 管理外なので、クローン直後は雛形から作る。**ここに書けるのは
`wrangler.jsonc` の `secrets.required` に宣言した名前だけで、宣言の無い名前は
書いても Worker に注入されない**（黙って `undefined` になる）。

Access の検証を飛ばす `ACCESS_DEV_BYPASS` を `secrets.required` に載せていないのは、
デプロイに含めたくないため。**あれは `pnpm dev` が `--var` で渡している**
（`package.json`）。渡らないと `/api/*` が全て 401 になる。バイパスは
**localhost 宛の要求にしか効かない**ので、デプロイ先に紛れ込んでも認証が
素通りすることはない（`src/lib/auth.ts`）。

**アカウント固有の値は追跡させない**（このリポジトリは公開）。`wrangler.jsonc` には
D1 の `database_id` も Access の値も書かない。全て `.prod.vars`（git 管理外）に置き、
`scripts/deploy-config.mjs` が設定（`wrangler.deploy.json`）と secret
（`.wrangler/deploy-secrets.env`）を組み立てて `pnpm deploy` が渡す。手順は README の
「デプロイ」を参照。

Node は 24 系。pnpm が未導入の環境で `corepack enable pnpm` が EACCES で失敗する場合は
`npm i -g pnpm` で入れてよい。

### 依存バージョンの制約

- **TypeScript は 5.9 に固定している。** vue-tsc 3.3.10 が TS 7 に未対応で、上げると
  `pnpm typecheck` が `ERR_PACKAGE_PATH_NOT_EXPORTED` で落ちる。vue-tsc の対応後に上げる
- `@cloudflare/vitest-pool-workers` 0.21 は `defineWorkersConfig` ではなく
  `cloudflareTest` プラグイン方式。`env` は `cloudflare:test` ではなく
  `cloudflare:workers` から import する
- テストから Worker の `fetch` を直接呼ぶときは `Request` ではなく `IncomingRequest`
  別名が必要（`src/api/health.test.ts` のパターンに従う）

### ローカル実行の細かい点

- **クローン直後は `pnpm build` を一度回さないと `wrangler dev` が起動しない。**
  `wrangler.jsonc` が Static Assets の置き場に `web/dist` を指しているので、
  無いと「The directory specified by the "assets.directory" field ... does not exist」で落ちる。
  開発中の画面は vite が配るため、ここで作った `dist` の中身は古くても構わない
- `pnpm dev` は **vite(5173) と wrangler dev(8787) の 2 プロセス構成**。画面は 5173 を開き、
  `/api` は vite の proxy で 8787 に渡る。本番は同一オリジンなので、クライアントは常に
  相対パスで `/api` を叩けばよい。`strictPort` は指定していないので、5173 が埋まっていれば
  vite は 5174 に逃げる（proxy 先の 8787 は固定なのでそのまま動く）
- **ローカル D1 のファイル名は `database_id` から決まる。** `.wrangler/state/v3/d1/` に
  `<hash>.sqlite` として置かれ、この hash が id ごとに違う。つまり **`wrangler.jsonc` の
  `database_id` を変えたり外したりすると、その瞬間から空の別 DB を見に行く。**
  手元のデータは消えていないが、古い方のファイルに置き去りになる（`database_id` を
  元の値に戻した設定で `--local` を引けば読み出せる）。実際、アカウント固有の値を
  git 管理外にした際に `database_id` ごと外したので、全ての開発機で一度これが起きた
- **`--local` に `wrangler.deploy.json` を使わない。** あちらには本番の `database_id` が
  入っているので、また別の空 DB が開く。ローカル作業は常に既定の `wrangler.jsonc`。
  `wrangler.deploy.json` は `--remote` 用（`pnpm db:migrate:remote` と `pnpm deploy`）
- **本番のデータを手元に持ってくる**なら
  `wrangler d1 export ratatoskr --remote -c wrangler.deploy.json --no-schema` で抜き、
  `INSERT INTO "feeds"` / `"entries"` の行だけを流し込む（`d1_migrations` と
  `sqlite_sequence` はローカルの方が正しい）。`wrangler d1 execute --file` は本文の長い
  記事で `SQLITE_TOOBIG` になるので、`.wrangler/state/v3/d1/<hash>.sqlite` へ
  sqlite3 で直接書く方が速い。書き込み中の競合を避けるため `pnpm dev` は止めておく

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kan/ratatoskr](https://github.com/kan/ratatoskr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
