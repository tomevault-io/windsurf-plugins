---
trigger: always_on
description: このファイルは AI エージェントが Vyline プロジェクトを理解しタスクを実行するための包括的なガイドです。
---

# AGENTS.md — Vyline エージェント向けガイド

最終更新: 2026-08-30

このファイルは AI エージェントが Vyline プロジェクトを理解しタスクを実行するための包括的なガイドです。

---

## プロジェクト概要

**Vyline** は LINE のサードパーティクライアントです。Bun + Hono + React で構築され、自前の LINE プロトコルスタック (`@vyline/protocol`) を持ちます。

- **目標**: LINE にログインし、メッセージの送受信・Flex/Rich 表示・テーマカスタマイズを行う
- **ライセンス**: MIT
- **ステータス**: Phase 0-3 完了。Beta 向けの UI・品質・配布準備とオープンチャット統合を継続中
- **外部依存**: `@evex/linejs` なし。Thrift 型は `@vyline/line-types`（vendored）

---

## 最初に読む順番

この repository は README、docs、submodule、workspace が多い。迷った場合は次の順番で読む。

1. `AGENTS.md`（このファイル）
2. `docs/README.md`
3. `docs/onboarding.md`
4. `docs/architecture.md`
5. 対象機能に一番近い docs
6. 実コード

README はユーザー向け入口。実装判断の正本にしない。

### AI が手間取りやすい点

- `Vyline/packages/protocol`、`Vyline/packages/plugin`、`Vyline/packages/themes`、`tools` は submodule/workspace の境界が見えにくい。まず `bun run vyl:doctor` で状態を確認する。
- docs整理では既存docsをテンプレートへ機械的に当て直さない。新規docsや大改修だけ `docs/templates/` を使い、既存docsは必要箇所だけ直す。
- README の正本は日本語が `README.src.md`、英語が `README.en.src.md`。`README.md` / `README.en.md` は生成物なので、原則として直接編集しない。README変更は両方の source に同じ内容を反映し、`bun run docs:readme` で生成して差分を確認する。
- README は元の構成を守る。新導線を入れる場合も、該当セクションへの追記に留める。
- 日本語と英語の README は情報量を揃える。機能、注意事項、パートナー、References、導入手順などを片方だけに追加しない。生成後は `README.md` と `README.en.md` の見出し・主要項目に欠落がないか確認する。
- `vyl doctor` / `vyl init` は軽い入口にする。npm publish、Docker build、Trivy container scan、full security scan は通常CLIに入れない。
- 重い処理は GitHub Actions の manual workflow、release workflow、schedule に寄せる。PRでは軽量チェックを優先する。

---

## 参照元・検索ツール

### 検索ツール: RPC_DICTIONARY

**`Vyline/packages/protocol/src/dictionary/rpcMap.ts`** — LINE.js 名 → Desktop 証拠 → Vyline 実装の対応表。

```ts
// 機能の実装場所を調べる:
// 1. rpcMap.ts で linejsName を検索
// 2. desktopEvidence で Desktop 内の実体を確認
// 3. stackApi → domainApi → backendApi の順に追跡
```

### 参考: @evex/linejs

`@evex/linejs` のメソッド名・構造パターンを参考にしていますが、依存はしていません。
RPC_DICTIONARY の `linejsName` フィールドが linejs との対応を示します。

### Desktop 解析ツール (Vyline-Search)

- `bun run vyline:check` — インストール版 / 実行中版 / 最新版の比較 (`--json`)
- `bun run vyline:latest` — Desktop 最新版バージョンの取得
- `bun run vyline:update [-- --unpack]` — LINE Desktop を最新版へ更新 (必要なら unpack も一括)
- `bun run vyline:versions` — インストール済みバージョン一覧 (`--json` 可)
- `bun run vyline:unpack` — Themida 保護された LINE.exe の unpack（LINE 稼働中は Frida 注入拒否されるため停止して実行）。`--version <ver>` でインストール済み過去版を明示選択可
- `bun run vyline:find-native -- <name>` — Desktop LINE.exe 内シンボル検索 (unpack → string scan → Ghidra)
- `bun run vyline:focus-recovered` — 逆コンパイル結果のキーワード分類
- `tools/` — スタンドアロンツール (Git Submodule として [vyline-search](https://github.com/nezumi0627/vyline-search) リポジトリをリンク)
- `Vyline/packages/plugin` — plugin-sdk + examples ([vyline-plugin](https://github.com/nezumi0627/vyline-plugin) を Submodule リンク)
- `Vyline/packages/themes` — VyTheme プリセット ([vyline-theme](https://github.com/nezumi0627/vyline-theme) を Submodule リンク)
- `source/desktop/` — 解析データ (gitignore)
- `docs/tools/` — ツール使用ガイド

---

## 現在のステータス

詳細ボード: **[docs/tasks/STATUS.md](docs/tasks/STATUS.md)** / 受け入れ条件: **[docs/tasks/PHASES.md](docs/tasks/PHASES.md)**

| Phase | 内容                                  | 状態        |
| ----- | ------------------------------------- | ----------- |
| 0     | Kickoff（docs）                       | done        |
| 1     | E2EE decrypt / send                   | done        |
| 2     | Docs / AGENTS / tasks                 | done        |
| 3     | Vyline + Desktop import + update-diff | done        |
| 4     | Telegram-like UI                      | in progress |
| 5     | Quality / perf                        | in progress |
| 6     | Beta 公開準備                         | in progress |

### 最近の主な変更 (2026-08-27)

- **Vyline Setup / アカウント設定**: 初回 3 ステップ設定、MID ごとの設定スキーマ、原子的 JSON 保存、進捗の復元を追加
- **引継ぎ / 診断**: 設定のみを含む SHA-256 検証 ZIP の import/export、サニタイズ済み診断ログの確認・出力・削除、GitHub Issue 作成導線を追加
- **セッション保護**: Windows のトークンを DPAPI(CurrentUser) で保護。サブデバイスをブラウザごとのランダムなインストール ID に結び付け、端末固有情報を保存しない
- **同期と表示**: 既読反映・未読位置・仮想リストのスクロールを安定化。アカウント切替時に前アカウントの UI 状態を残さない
- **遠隔利用**: LAN 公開を既定で無効にし、Tailscale の検出と URL 表示を追加

### 最近の主な変更 (2026-08-18)

- **リブランディング**: `@vyline/nezuline` → `@vyline/protocol`、`Nezu*` → `Vyline*`。旧 `nezu-*.json` / `nezuline` データディレクトリからの自動移行
- **VylineBackup**: 設定 > VylineBackup からトーク履歴・メディアのスナップショットを作成/復元/削除（`data/backups/`）。復元は「すべて / チャット選択」「メディア含む / テキストのみ」を選択可
- **チャット詳細ログ**: 送受信・アナウンス（CHATEVENT）をタイミング付き JSONL で記録（`data/logs/`）。画像・動画・音声・ファイル・スタンプのメディア情報も記録。設定 > 詳細・復元 > デバッグログで閲覧
- **高画質送信**: 表示タブに「高画質で画像送信」トグル（圧縮せず元画質で送信）
- **useVirtualList**: 実測高さ変更時にオフセットを再計算するよう修正
- **ブロックリスト**: キャッシュ + background キュー + 8s タイムアウトで 504 回避

### 最近の主な変更 (2026-08-17)

- **メンション**: `@ALL` / `@名前` 送受信（`contentMetadata.MENTION` の `MENTIONEES` 形式、Desktop 準拠）。入力時 `@` で候補ピッカー、表示はハイライト + アイコン
- **LINE 絵文字**: チャット一覧・返信引用で `￼` プレースホルダが表示される問題を修正
- **Flex**: カルーセルのマウスドラッグ、wrap テキストのクリップ修正
- **画像送信**: クライアント側で 2048px JPEG 圧縮。E2EE 鍵整備のキャッシュ化 + メディア送信 90s タイムアウト。`isMissingGroupKeyError` の判定追加
- **画像表示**: 自送信 E2EE メディアを `contentMetadata.keyMaterial` で直接復号する高速パス（履歴 RPC を飛ばす）
- **設定**: 詳細・復元に「設定を初期化」（ログイン状態・履歴は保持）

### 過去の変更 (2026-07-31)

- プロフィール/メンバー表示: API タイムアウト追加、MID短縮表示、空配列上書き防止、キャッシュ汚染防止
- グループ作成: 禁止解除オプション追加 (自己責任)
- チャット同期: 手動同期ボタン追加、visibility change 時自動差分同期
- E2EE/メディア: グループ鍵不在時 E2EE スキップ、重複呼び出し抑止、USER chat 誤呼び出し防止
- スタンプ表示: プロキシURL判別修正
- 招待: u* MID 検証・フィルタリング

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nezumi0627/vyline](https://github.com/nezumi0627/vyline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
