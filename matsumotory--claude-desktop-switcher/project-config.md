---
trigger: always_on
description: このファイルは Claude Code がセッション開始時に自動ロードする、本リポジトリの **一次正典** です。プロジェクトの規約・運用ルールはすべて本ファイルに集約します（旧 `.agents/AGENTS.md` の内容を取り込み済み。AGENTS.md は本ファイルへの薄いポインタに縮約しています）。横断ルールはマシン全体のグローバル `~/.claude/CLAUDE.md` を、デザイン等の具体手順は `.agents/skills/<name>/SKILL.md` を一次情報とします。
---

# CLAUDE.md

このファイルは Claude Code がセッション開始時に自動ロードする、本リポジトリの **一次正典** です。プロジェクトの規約・運用ルールはすべて本ファイルに集約します（旧 `.agents/AGENTS.md` の内容を取り込み済み。AGENTS.md は本ファイルへの薄いポインタに縮約しています）。横断ルールはマシン全体のグローバル `~/.claude/CLAUDE.md` を、デザイン等の具体手順は `.agents/skills/<name>/SKILL.md` を一次情報とします。

## 0. Skill-First Gate（最優先・例外なし）

デザイン/視覚やドメイン作業に着手する前に、**同じターン内・編集の前に**、(1) 該当する `.agents/skills/<name>/SKILL.md` を `Read` し、(2)「どのスキルのどのルールをこの変更に当てるか」を 1 行で明示してから編集する。

- **対象**: アプリ UI・CSS・レイアウト・余白・配色・タイポグラフィ・アイコン・コピー/マイクロコピー・LP・スクリーンショットなど、デザイン/視覚に関わるすべて。加えて各ドメイン作業（Rust 実装・Tauri・バグ修正・仕様変更・コミット・PR レビュー・整合性監査・日本語タイポ等）は下の Skill トリガーテーブルに従う。
- **「小さい/自明だから」は免除理由にならない**。フッターの寄せ・スウォッチの形・選択ハイライト・左バー——まさにこの「小さい変更」を雰囲気でやって繰り返し外している。サイズや明白さに関係なくゲートを通す。
- **自己捕捉**: スキルを引かずに着手してしまったと気づいたら、即停止・違反を自己申告し、スキルを Read して引用したルールでやり直す。ユーザーに指摘される前に自分で直す。
- **選び方**: 日本語の UI/LP/見出しは `japanese-typography-qa`、フロント/LP デザインは `design-taste-frontend` / `high-end-visual-design` / `minimalist-ui` 等。用途別の選び方はグローバル `~/.claude/CLAUDE.md` と各スキルの説明を参照。
- **強制の仕組み（意志力に頼らない）**: `.claude/settings.json` の PreToolUse フック `.claude/hooks/skill-first-reminder.sh` が、`website/`・`docs/`・`README`・`crates/desktop/ui/`・`*.html/*.css/*.md` を Edit/Write する直前に、本ゲート（該当スキルを読む）と全サーフェス伝播（`propagate-changes-to-all-surfaces`）・変更後の `/audit-consistency`・スクショ再生成（`scripts/appshot`）を自動リマインドする。フックの注意が出たら従う。指摘される前に自分でゲートを通す。
- **ソフトなリマインダは willpower 依存で漏れる（v0.23.0 の実例）**: 上のフックは発火していても、長い実装セッションの後半で無視されうる（`crates/desktop/ui` のボタン文言を変えたのに出荷スクショを再生成せず、指摘されて初めて確認した）。加えて、使い捨ての headless キャプチャで描画を確かめただけで「GUI 検証済み」と取り違えない。出荷される `website/assets` のスクショを実際に再生成することが検証であり、別物の一時キャプチャはその代わりにならない。だから機械で止めるハードなゲートを併用する: `crates/desktop/ui/{index.html,main.js,style.css}` を変えた PR が `website/assets/*.png` を更新していないと CI ワークフロー `Verify screenshots`（`.github/workflows/verify-screenshots.yml`）が落ちる。純粋に非視覚の UI 変更のときだけ、コミットメッセージに `Skip-appshot: <理由>` 行を足して明示的に外す（このゲート自体を無効化して回避しない）。
- **スキルを読んだら、まず「このタスクに効く出荷前チェックと伝播先」を明示タスクに落とす（着手前・毎回）**: 失敗の型は、最初にスキル/ルールを見ても、作業に没頭したり追加のサブ作業が出た後半で適用を忘れること（matsumotory 2026-07-09「作業に夢中になったり追加作業が出た時に見忘れる。スキルを見た時にまず計画を書くのがいい」）。対策として、該当スキルを Read した直後に、そのスキルの出荷前チェックリストとこの変更の伝播先（アプリ UI・トレイ・docs ja/en・LP ja/en・スクショ・OG 等）を `TaskCreate` か書き出した計画に列挙し、完了報告の前に 1 項目ずつ実測で消し込む。記憶や「さっき読んだ」に頼らない。**作業中に増えたサブ作業にも同じチェックを再適用する**。上のハードなゲートは最後の砦、この計画化は作業中の予防で、両方を併用する。
- この Gate はグローバル `~/.claude/CLAUDE.md` を一次正典とし、本ファイルで二重化する。

## プロジェクト概要

Claude Desktop Switcher (CSW) は、Claude Desktop アプリ (GUI) と Claude Code (CLI) のプロファイル (データディレクトリ + macOS Keychain) を安全に隔離・共有する Rust + Tauri v2 製の macOS アプリ。

- `crates/core` (`csw-core`): ビジネスロジック (OS パス抽象化・Keychain 操作・プロファイル/シンボリックリンク管理)
- `crates/cli` (`csw`): clap ベースの CLI
- `crates/desktop` (`csw-desktop`): Tauri v2 GUI (システムトレイ常駐)

正典ドキュメント: [docs/SPECIFICATION.md](docs/SPECIFICATION.md) / [docs/USER_GUIDE.md](docs/USER_GUIDE.md) / [docs/USER_GUIDE_EN.md](docs/USER_GUIDE_EN.md)。LP は `website/`。

## ビルド・テスト・検証コマンド

```bash
cargo build --workspace            # 全クレートビルド
cargo test --workspace             # テスト (core はテスト時 MockKeychainProvider を使用)
cargo clippy --workspace --all-targets -- -D warnings   # lint
cargo fmt --all                    # フォーマット
```

GUI の実機確認は `cargo tauri dev` (要 `cargo install tauri-cli`)。デスクトップ署名/公証/DMG は GitHub Actions の `Release Please` ワークフローが担う。リリースの実施手順 (release-please の release PR を `--admin` を使わずマージし、署名・公証つき DMG と `csw` の公開まで監視する正規手順) は [.agents/skills/core_pr_merge_checklist/SKILL.md](.agents/skills/core_pr_merge_checklist/SKILL.md) の「リリース PR (release-please) のマージ」節を参照する。リリースは同節の「リリース前の正しさ検証」を通した上で、確認を取らず自律的に行う。

LP (`website/`) のプレビューは Claude の launch 機能で `.claude/launch.json` の `lp` を起動する (preview_start で `website/` を `python3 -m http.server` 配信、EN は `/`、日本語は `/ja/`)。`website/` をルートに配信しないと相対パス (`../style.css` 等) が解決しないので、単体 HTML を file:// で開くのではなくこの設定で見る。

**LP (GitHub Pages) の公開は恒久許可済み**: `website/` の変更が `main` に入ったら、公開の確認を取らずに公開してよい。`pages.yml` は `workflow_dispatch` のみで push では自動反映されないため、マージ後に `gh workflow run pages.yml` を実行し、run が success になるまで監視する。公開後はライブサイト (`curl` 等) で反映を実地確認する (GitHub Pages / ブラウザキャッシュがあるため「デプロイ成功」だけで済ませない)。この恒久許可は CSW の LP に限る。

## Skill トリガーテーブル

以下の条件に合致したら、該当する `.agents/skills/<name>/SKILL.md` を `Read` で読み、その手順に従う。`/bugfix` `/spec-first` `/audit-consistency` はスラッシュコマンドからも起動できる (`.claude/commands/`)。

| 発動条件 | スキル |
|---|---|
| 派生サブタスク (spawn_task / Agent 委譲 / 別 PR 化 / CI 待ち中の並行作業 / 割り込み) を始めるとき | `core_worktree_for_derived_tasks` |
| 新規実装・大幅修正の着手前、セッション開始/終了時 (Plan を `docs/proposals/` でバトンパス) | `core_session_handoff` |
| 日本語 UI / LP / ドキュメントのコピー・フォントサイズ・折り返し・行間・見出しスケールに触れるとき | `japanese-typography-qa` |
| 「完了/PASS」と報告する前・PR 提出前・リファクタ/名称変更後 (検証順序とエビデンス強制) | `core_qa_process` |
| PR をマージする / マージ可否を判断するとき | `core_pr_merge_checklist` |
| CI (GitHub Actions) が 10 分以上ハングしている疑いのとき | `core_ci_hang_recovery` |
| 複数 crate / 複数ファイルにまたがる作業を役割分担で進めるとき | `core_agent_roles` |
| 多段の調査・実装・検証を並列化したいとき (Workflow / サブエージェント) | `core_ai_workflow` |
| バグ修正に着手するとき (RED テストファースト) | `core_bug_fix_protocol` (`/bugfix`) |
| 新機能・仕様変更に着手するとき (仕様合意→RED→GREEN) | `core_spec_first_development` (`/spec-first`) |
| git commit する直前 | `core_commit_standard` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matsumotory/claude-desktop-switcher](https://github.com/matsumotory/claude-desktop-switcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
