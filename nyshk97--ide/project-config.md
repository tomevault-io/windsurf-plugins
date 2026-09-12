---
trigger: always_on
description: このリポジトリで Claude Code が開発を続けるためのガイド。`~/.claude/CLAUDE.md`（グローバル）と併せて読まれる。
---

# CLAUDE.md

このリポジトリで Claude Code が開発を続けるためのガイド。`~/.claude/CLAUDE.md`（グローバル）と併せて読まれる。

ユーザーから明示の指示がない限り、ここに書いてあるルールが優先する。

---

## このプロジェクトは何か

**PolePole**: cmux + Ghostty + yazi + git-watch + Claude Code を 1 つに統合した自作 IDE（macOS 専用）。
2026-05-23 にプロジェクト名を `ide` から `PolePole` にリネーム済み。技術文脈は小文字 `polepole`、ブランド表記は CamelCase `PolePole`。リポジトリ名は歴史的事情で `nyshk97/ide` のまま。

要件は [REQUIREMENTS.md](./REQUIREMENTS.md)。実装の進捗とアーキ概要は:

- 概要: [README.md](./README.md)
- モジュール構成: [docs/ARCHITECTURE.md](./docs/ARCHITECTURE.md)
- 開発手順: [docs/DEV.md](./docs/DEV.md)
- 動作確認: [VERIFY.md](./VERIFY.md)
- 残タスク: [docs/BACKLOG.md](./docs/BACKLOG.md)

---

## 何かを始める前に必ず読む

1. **要件と整合する変更か** — `REQUIREMENTS.md` のセクション番号で議論する
2. **plan があるか** — `docs/plans/` の進行中 plan があれば、ステップ通りに進める
3. **テスト用フラグの位置** — `POLEPOLE_TEST_*` 環境変数の一覧は `docs/DEV.md`

---

## ビルドと動作確認

詳細は [docs/DEV.md](./docs/DEV.md)。最低限:

```bash
mise run build                                        # ビルド（regen を含む）
./scripts/polepole-launch.sh                          # 起動（kill + open）
./scripts/polepole-screenshot.sh /tmp/v.png           # フロントウィンドウだけ撮影
./scripts/polepole-keystroke.sh --enter "echo hello"  # キーストローク送信
```

確認手順は [VERIFY.md](./VERIFY.md) の番号付きセクションを「変更内容に関係するものだけ」実行する（毎回全部やらない）。

---

## 動作確認は手抜きしない

修正後にユーザーへ確認を求める前に、自分で動作確認を行うこと。

- **コードの確認**: `mise run build` が通る
- **UI の確認**: `./scripts/polepole-launch.sh` + `./scripts/polepole-screenshot.sh` で画面を取って自分で確認する
- **テスト用フラグを活用**: `POLEPOLE_TEST_AUTO_ACTIVATE_INDEX` `POLEPOLE_TEST_AUTO_PREVIEW` `POLEPOLE_TEST_AUTO_FULLSEARCH` `POLEPOLE_TEST_TOAST` で起動時に状態を仕込んで screenshot 取得まで自動化できる
- **Debug 版へ `POLEPOLE_TEST_*` を渡す UI 検証は `open -n "$APP" --env KEY=value` を優先**: PolePole Release 内の Claude/Codex から Debug binary を直叩きすると、親プロセスの `GHOSTTY_RESOURCES_DIR` などを継承して Release 側 resource を参照し、window / screenshot 検証が不安定になることがある。環境変数依存そのものを検証したい場合だけ binary 直叩きを使う
- **キーストローク・フォーカス移動が要る検証は `POLEPOLE_TEST_EVENT_FILE` の注入フックで自動化する**（Debug 限定。`docs/DEV.md` の「テスト用環境変数」、実例は `scripts/verify-find-bar-focus.sh`）。`polepole-keystroke.sh` 系（osascript の補助アクセス）は PolePole 内 Claude Code からは `login` 介在で効かないので使わない。`polepole-screenshot.sh`（画面収録）は OK。マウスクリック（「読み込む」ボタン押下後の挙動・Markdown のローカルリンククリック等）は注入フック未対応なのでユーザーに目視依頼する
- **Dock 検証では Release/Dev の取り違いに注意**: Brew 版 (`PolePole`) と Debug 版 (`PolePole Dev`) が両方 Dock にあるとき、AppleScript で `UI elements whose name contains "PolePole"` を使うと両方マッチして取り違える。Dev 版だけ欲しいときは `name is "PolePole Dev"` で完全一致させる。同様に `screencapture -R<x,y,w,h>` で Dock アイコン領域を撮る場合も、位置を取り違えると「Dev 側を変更したのに古い」と誤判定する

「確認しました」だけで済ませず、実行コマンド・出力（抜粋）・pass/fail 判定を報告する。

### ⚠️ PolePole の中で検証するには PolePole.app に TCC 権限が要る

`polepole-screenshot.sh` / `polepole-launch.sh` を PolePole 内ターミナルの Claude Code から回すには、`/Applications/PolePole.app` に **画面収録** と **フルディスクアクセス**（`~/Library/CloudStorage/` 配下の dotfiles を読むため）が付与されている必要がある。剥がれていると「could not create image from display」「`.zshrc` が読めずデフォルトプロンプト・mise/`claude` が PATH に無い」になる。Release ビルドは安定した Developer ID 署名なので brew 更新では剥がれない。詳細・再付与手順は [docs/DEV.md の「TCC（プライバシー）権限の罠」](./docs/DEV.md#tccプライバシー権限の罠)。

`ide` → `PolePole` リネーム時は Bundle ID が変わるため、旧 IDE.app に付与していた TCC 権限は新 PolePole.app には引き継がれない。初回は System Settings から手動で再付与する。

### ⚠️ Brew 版データ (`polepole/`) は触らない。検証は `polepole-dev/` で

`~/Library/Application Support/polepole/projects.json` には**Brew 配布版 (Bundle ID `local.d0ne1s.polepole`) でユーザーが手で pin したプロジェクト一覧**が入っている。Debug ビルドは Bundle ID が `local.d0ne1s.polepole.dev` に分離されているので、`mise run build` → `./scripts/polepole-launch.sh` 由来の起動・検証では `~/Library/Application Support/polepole-dev/projects.json` 側に書かれ、Brew 版データには触らない。

VERIFY.md の検証手順は固定フィクスチャで `polepole-dev/projects.json` を上書きする → `rm -f` する流れなので、Dev 版でもピン留めを残したい運用なら念のためバックアップ:

```bash
# 検証開始前
BACKUP_DIR=$(mktemp -d)
cp -a "$HOME/Library/Application Support/polepole-dev/" "$BACKUP_DIR/polepole-dev-backup" 2>/dev/null || true

# 検証完了後
rm -rf "$HOME/Library/Application Support/polepole-dev"
mv "$BACKUP_DIR/polepole-dev-backup" "$HOME/Library/Application Support/polepole-dev" 2>/dev/null || true
```

**Release configuration を直接起動して検証するときは `polepole/` 側を扱うことになる**ので、その経路では引き続き `polepole/` を退避してから検証する。

過去に Bundle ID 分離前のビルドで旧 `ide/` を破壊したインシデントあり（2026-05-09）。分離後はこの経路は塞がっているが、Release 検証時の警告は変わらず有効。

---

## SwiftUI / Swift 6 の落とし穴（既出）

[docs/DEV.md の同セクション](./docs/DEV.md#swift-6-strict-concurrency-の落とし穴) にまとまっている。**新しく踏んだら追記する**。

代表例:
- AppleScript の `click at {x, y}` は SwiftUI の `onTapGesture` に届かないことがある → `POLEPOLE_TEST_*` で迂回
- `Ctrl+M` の判定は `keyCode == 46`（characters は CR にマップされる）
- `URL` の `==` は scheme/baseURL の差で一致しないことがある → `URL.standardizedFileURL.path` を String キーに
- Debug ビルドは PRODUCT_NAME=`PolePole Dev` なので `.app` / プロセス / バイナリすべてに空白を含む。動作確認スクリプトでは `pkill -x "PolePole Dev"` / `pgrep -f "PolePole Dev.app/Contents/MacOS/PolePole Dev"` / AppleScript の `tell process "PolePole Dev"` のように毎回クオートする
- `NSViewControllerRepresentable` を別の `NSViewControllerRepresentable` の中にネストするとクラッシュする（SwiftUI の VC 親子ツリーが壊れる）。複数ペインを AppKit で組み合わせるときは単一の `NSViewController` サブクラスの中で直接 `NSSplitView` を管理する

---

## libghostty (Metal renderer) の制約


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nyshk97/ide](https://github.com/nyshk97/ide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
