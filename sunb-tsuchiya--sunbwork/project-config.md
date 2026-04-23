---
trigger: always_on
description: このファイルはすべての会話の開始時に必ず参照すること。コードの作成・修正はここに記載されたルールに従う。
---

# CLAUDE.md - SunBWork プロジェクト ルール

このファイルはすべての会話の開始時に必ず参照すること。コードの作成・修正はここに記載されたルールに従う。

---

## Claude へのワークフロー指示

- Vue / JS ファイルを変更したら、必ず最後に `npm run build` を実行すること（許可済み）。
- `npm run build` はプロジェクトルート（`/home/w229/SunBwork`）で実行する。
- Artisan は必ずコンテナ内: `docker compose exec laravel bash -lc "php artisan ..."`
- **Docker が使えない場合はホスト上の npm（node v24, npm v11）で直接 `npm run build` を実行する。**

### public/build/assets が root 所有になる問題

Docker コンテナ内でビルドすると `public/build/assets/` が root 所有になり、次回のホストからの `npm run build` が `EACCES Permission denied` で失敗する。
対処: `sudo chown -R $USER:$USER public/build/ && sudo chmod -R 755 public/build/assets`

### ユーザーから「gitにアップ」「さくらにデプロイ」を求められたときの手順

① `git status --short | grep -v "public/build"` で未コミット確認（Controller/Model/Migration/routes 漏れに注意）

② routes/web.php 変更があれば Ziggy 再生成:

```bash
docker compose exec laravel bash -lc "php artisan ziggy:generate resources/js/ziggy.js"
```

③ さくら用ビルド（`.env` の VITE_APP_BASE_PATH は **2箇所** ある）:

```bash
sed -i 's/^VITE_APP_BASE_PATH=$/VITE_APP_BASE_PATH=\/members/' /home/w229/SunBwork/.env
npm run build
```

④ コミット:

```bash
git add <変更ファイル> public/build/ resources/js/ziggy.js
git commit -m "feat/fix/build: ..."
```

⑤ **コミット直後** に .env をローカル用に戻してローカルビルドも実行（コミット不要）:

```bash
sed -i 's/^VITE_APP_BASE_PATH=\/members$/VITE_APP_BASE_PATH=/' /home/w229/SunBwork/.env
npm run build
```

> ⚠️ 重要: ⑤ を実行しないとローカル開発環境の .env が本番（さくら用）のままになり、次回の開発ビルドで `/members` ベースパスが適用され、ローカル環境が壊れます。必ずコミット直後に実行してください。

⑥ ユーザーへ伝える:

```
【あなたの操作が必要です】
1. git push origin main
2. さくら SSH: cd ~/SunBWork && git pull && php artisan migrate && php artisan config:clear
   ※ マイグレーションがない場合は migrate は省略可
```

---

## プロジェクト概要

**SunBWork** は Laravel + Inertia.js (Vue 3) + Vite + Tailwind CSS で構築された SPA 型の業務管理システム。

- ロール: SuperAdmin / Admin / Coordinator / Leader / User
- 機能: ProjectJob 管理・割当（JobBox/MyJobBox）、カレンダー、チャット、日報、ワークロード解析
- DB: MySQL / Sanctum + Cookie SPA 認証 / Laravel Echo (WebSocket)

**主要フォルダ:**

```
app/Http/Controllers/
  Admin/  Coordinator/  Leader/  ProjectJobs/  User/  Bot/  Chat/  Diaries/
app/Models/
database/migrations/          # backups/ は読み飛ばす
resources/js/
  Pages/                      # Inertia ページ（ロール別サブディレクトリ）
  Components/                 # 大文字始まり = プロジェクト固有
  components/ui/              # 小文字始まり = shadcn/ui 系
  layouts/AppLayout.vue       # メインレイアウト（全ページ共通）
  ziggy.js                    # Ziggy ルート定義
routes/web.php                # SPA ルート（api.php には置かない）
z_instructions/               # 詳細ドキュメント（backups/ は読み飛ばす）
```

---

## UI / レイアウト ルール（最優先）

AppLayout は `py-12 > max-w-7xl` を内部に提供済み。ページ側はデフォルトスロットに直接カードを置く:

```vue
<AppLayout title="ページタイトル">
  <template #header><h2>見出し</h2></template>
  <div class="rounded bg-white p-6 shadow">
    <!-- コンテンツ -->
  </div>
</AppLayout>
```

**NG:** `<main>` タグ、`py-2/py-12` の重複ラップ、`mx-auto max-w-7xl` の重複ラップ、`shadow-xl sm:rounded-lg`

**AppLayout スロット:** `#header` / `#headerExtras` / `#tabs` / デフォルト

**AppLayout が provide する値:** `authUser`（ログインユーザー）/ `user`（ページの user prop）

**ToastUnified は AppLayout 内にグローバル配置済み。各ページで重複させない。**

**ロール別カラー:** SuperAdmin=黄 / Admin=赤 / Leader=オレンジ / Coordinator=緑 / User=青

**テーブル:** `min-w-full divide-y divide-gray-200`、ヘッダ `bg-gray-50`、行 `hover:bg-gray-50`

**Ziggy の route() 使用時はパラメータ名をオブジェクトで渡す:**

```js
route('coordinator.project_jobs.show', { projectJob: job.id });
```

---

## ロール共有ページのルーティングルール

AppLayout.vue の `currentRouteContext` computed はルート名プレフィックスでタブを決定:

- `superadmin.*` → superadmin / `admin.*` → admin
- `leader.*` or `workload_setting.*` → leader
- `coordinator.*` or `project_jobs.*` → coordinator
- それ以外 → 必ず `'user'` を返す（user_role にフォールバックしてはいけない）

複数ロール共有ページでは `routePrefix` computed（Vue）と `routePrefix()` メソッド（PHP）でロール別にルートを解決する。

**Clients テーブルの注意:** DBカラムは `notes`。フォームフィールド名 `detail` との乖離に注意。

---

## project_job_assignments 統合テーブル設計（重要）

### 設計概要

`project_job_assignments` が **JobBox・MyJobBox 両方の唯一のテーブル**。
`project_job_assignment_by_myself` テーブルは migration 500001 で統合・削除済み。

**区別ルール:**

- `sender_id = user_id` → **自己割当**（旧 project_job_assignment_by_myself）
- `sender_id ≠ user_id` or NULL → **Coordinator 割当**

**スコープ（ProjectJobAssignment モデル）:**

```php
scopeSelfAssigned()        // whereColumn('sender_id', 'user_id')
scopeCoordinatorAssigned() // sender_id != user_id OR sender_id IS NULL
```

**統合時に追加されたカラム:** `completed` / `scheduled` / `scheduled_at` / `read_at` / `start_time`

**`desired_start_date` は存在しない（意図的に除外）。**
**`WorkloadAnalyzerController` など `project_job_assignments` に対して期間フィルターをかける場合は `desired_end_date` を使うこと（`desired_start_date` を参照すると Column not found エラー）。**

### ProjectJobAssignmentByMyself モデル

`ProjectJobAssignment` を継承したエイリアス（後方互換のため残存）。
`$table` は親クラスの `'project_job_assignments'` を使用（**旧テーブルは参照しない**）。
`booted()` で `whereColumn('sender_id', 'user_id')` のグローバルスコープを自動適用。
**新コードでは `ProjectJobAssignment::selfAssigned()` スコープを使うこと。**

> **migration 2026_04_03_300001** により、旧テーブル `project_job_assignment_by_myself` の
> 全データ（48件）を `project_job_assignments` へ移行済み。`sender_id = user_id` が自己割当マーカー。
> 旧テーブルはバックアップとして残存しているが、アプリからは参照しない。
> `events.project_job_assignment_id` も新テーブルのIDに更新済み。

### JobBox vs MyJobBox の違い

| 機能     | データソース                                                   | 完了ルート                      |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SunB-Tsuchiya) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
