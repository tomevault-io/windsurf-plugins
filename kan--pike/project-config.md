---
trigger: always_on
description: **Pike** は Tauri v2 (Rust + Vue/TypeScript) で構築する軽量開発環境。第一ターゲットは Windows で、
---

# Pike — Claude Code ガイド

## プロジェクト概要

**Pike** は Tauri v2 (Rust + Vue/TypeScript) で構築する軽量開発環境。第一ターゲットは Windows で、
macOS はローカルのシェルで開発できるところまで対応する。
「AI エージェント × ターミナル」に特化し、VS Code より大幅に軽いことが最重要の差別化点。

### 設計思想
- **軽さ最優先**: Monaco は使わない。CodeMirror 6 のみ。拡張機能システムは作らない
- **タブ統一**: エディタ・ターミナル・Docker logs をすべて同一タブで扱う
- **Rust はステートレスに**: Rust は I/O ブリッジに徹する。セッション復帰は各ツールの resume 機能（`claude --continue` 等）に委譲
- **外部依存は明示**: rg なければ grep、と graceful degrade する。tmux はオプション機能

### ターゲット環境
- **OS**: Windows 11（メイン開発・動作環境）。macOS はローカルのシェルで開発できるところまで対応する（詳細と制約は `.claude/rules/platform.md`）
- **実行環境**: WSL2 上のシェル・Docker コンテナ、または Windows ホスト上のシェル。macOS ではホストのログインシェル
- **GUI**: Tauri v2 webview（ホストのネイティブプロセス。Windows は WebView2、macOS は WKWebView）
- **対応シェル**: WSL bash / cmd.exe / PowerShell / Git Bash / ローカル Unix シェル（macOS）

---

## 詳細ルール（触る領域のものを読む）

このファイルには全体像と運用を置き、領域別の実装ルールは `.claude/rules/` に分けてある。どれも非自明な判断・定数・落とし穴の記録で、読まずに書くと過去に踏んだものを踏み直す。

各ファイルは先頭の `paths:` フロントマターで対象を絞ってあり、**当てはまるファイルを Read したときにだけ読み込まれる**（起動時には読み込まない。全部を常に読み込むと指示ファイルの合計が上限を超える）。**Grep や Glob だけで済ませて Read しないまま書くときは、下の表から該当するものを自分で読むこと。** ルールを足したり、担当するファイルが増えたりしたら `paths:` も直す。

| ファイル | 中身 |
|---|---|
| `rust.md` | Tauri コマンドの形・状態管理・文字列・非同期・命名規約 |
| `rust-window.md` | Rust からウィンドウと webview を触るとき（`Window` を使う・ack を待たない・ドロップの無効化） |
| `frontend.md` | どの画面にも当てはまる規則（Vue/Pinia の構成・URL を開く・overlay・フォーカス・スタイル・アイコン・i18n・禁止事項） |
| `tabs.md` | タブ管理・タブバー・作業領域の分割・OS ファイルドロップ |
| `settings-ui.md` | 設定画面・テーマの追従・フォント・シェルプロファイル |
| `testing.md` | 自動テストの範囲・検証バイナリの置き場 |
| `terminal.md` | PTY とシェル対応・ターミナルの coding agent 補助・ターミナル検索 |
| `shortcuts.md` | キーボードショートカットの層と正本・ターミナルとのキーの取り合い・マニュアルとの照合 |
| `project.md` | プロジェクトの登録・管理・切替・一時プロジェクト・セッション永続化 |
| `sync.md` | 設定とプロジェクト一覧の同期（#403） |
| `window.md` | マルチウィンドウ・ウィンドウ状態の永続化・仮想デスクトップ・背景透過 |
| `os-integration.md` | グローバルモード・`pike` CLI・`--wait`・ジャンプリスト・トレイ |
| `git.md` | git CLI ブリッジ・コンフリクト解消・ログ・リモート操作・worktree |
| `git-diff.md` | diff タブと履歴タブ |
| `git-graph.md` | ブランチグラフとコミットタブ |
| `editor.md` | ファイルツリー・エディタ本体・保存の責任・QuickOpen・定義ジャンプ・アウトライン |
| `preview.md` | Markdown の入力支援・各種プレビュー・画像ビューワと PDF |
| `search.md` | 検索パネル・rg の検出・置換 |
| `watcher.md` | ファイル監視 |
| `issues.md` | issue パネルと issue タブ |
| `panels.md` | 診断パネル（Problems）とタスクランナー |
| `agent.md` | エージェントの一覧と起動・シェルへの問い合わせ（エージェントはターミナルで動かす、#275） |
| `agent-hook.md` | 入力待ちの知らせ・デスクトップ通知・hook による申告・`CLAUDE_CONFIG_DIR` |
| `agent-usage.md` | トークン使用量とレート制限の表示 |
| `docker.md` | bollard 連携・compose の探索・ログ・ポートフォワード |
| `build.md` | 開発ビルド・本番ビルド限定の落とし穴（CSP）・E2E スクリーンショット・CI・セルフアップデート |
| `platform.md` | Windows / macOS の分岐の作法・GUI プロセスの PATH・macOS で持たない機能・rg サイドカー |

---

## アーキテクチャ

```
┌─────────────────────────────────────────────────────────┐
│  Tauri WebView (ネイティブプロセス)                      │
│  ┌────────────┐  ┌──────────────────────────────────┐  │
│  │ 左サイドバー│  │ タブペイン                        │  │
│  │ アイコン   │  │ [📌CC][📌Codex][editor][shell][+]│  │
│  │ ナビ       │  │                                    │  │
│  │ ─────────  │  │  xterm.js / CodeMirror 6          │  │
│  │ 🗂 files   │  │  (アクティブタブのコンテンツ)      │  │
│  │ 🌿 git     │  │                                    │  │
│  │ 🔍 search  │  └──────────────────────────────────┘  │
│  │ 🐋 docker  │                                         │
│  │ 📁 projects│                                         │
│  │ 📋 tasks   │                                         │
│  │ 🔭 outline │                                         │
│  │ ⚠ problems │                                         │
│  │ ✅ issues   │                                         │
│  └────────────┘                                         │
└──────────────┬──────────────────────────────────────────┘
               │ Tauri IPC (invoke / events)
┌──────────────▼──────────────────────────────────────────┐
│  Rust バックエンド                                        │
│  pty_manager   git_manager   fs_watcher   search         │
│  project_store docker_client                             │
└──────────────┬──────────────────────────────────────────┘
               │ wsl.exe spawn / ホストのシェル / bollard / git CLI / notify
┌──────────────▼──────────────────────────────────────────┐
│  WSL2 (Windows) / ホストのログインシェル (macOS)          │
│  Claude Code / bash / zsh / etc.                         │
│  Docker (WSL2 backend) ← コンテナ群                      │
└─────────────────────────────────────────────────────────┘
```

---

## ディレクトリ構成

ファイル単位の構成は `.claude/structure.md` にある（起動時には読み込まない）。どこに何があるかを
探すとき、新しいモジュールの置き場を決めるときに読む。**`src/` と `src-tauri/src/` にファイルを足したら
そこにも 1 行足す**（`just check-docs` が照合する）。

```
pike/
├── CLAUDE.md / README.md / justfile
├── docs/manual/        # ユーザーマニュアル（画像は img/）
├── scripts/            # bump・check-docs・check-shortcuts・rg の取得など
├── tests/              # フロントの純粋なロジックのテスト
├── src-tauri/src/      # Rust バックエンド
├── src/                # Vue/TypeScript フロント
└── .claude/            # rules/（領域別ルール）・skills/release/（リリース手順）・structure.md
```

---

## 開発の進め方

機能追加・修正は **GitHub Issue 駆動**で行う。作業前に対象 Issue を確認すること。

ドキュメントの役割分担を守ること:

- **README.md** … ユーザー向け（概要・インストール・主な機能・マニュアルへの導線）。AI 開発の内部情報は書かない。
- **docs/manual/** … ユーザーマニュアル（日本語）。使い方・操作手順はここに集約し、拡充する。
- **CLAUDE.md（本ファイル）** … AI 開発のための情報のうち、全体像・構造・規約・運用。ユーザー向けの使い方は書かない。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kan/pike](https://github.com/kan/pike) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
