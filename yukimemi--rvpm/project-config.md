---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## コンセプト

- **Extremely Fast**: Rust の並行処理 (Tokio) と merge 済みディレクトリ + 事前コンパイル済み loader.lua による爆速起動。
- **Type Safe & Robust**: TOML ベースの設定を serde で型付け、`resilience` 原則で 1 プラグインの失敗が全体を止めない。
- **Convention over Configuration**: `{config_root}/<host>/<owner>/<repo>/` 配下の `init.lua` / `before.lua` / `after.lua` を規約に従って自動読み込み。
- **Hybrid CLI**: 引数による一発操作 + `FuzzySelect` / TUI によるインタラクティブ操作を両立。
- **Pre-compiled loader**: `vim.go.loadplugins = false` で Neovim の plugin loading を無効化し、generate 時に静的な loader.lua を生成。merge 最適化と事前 glob で起動時 I/O を削減。

## Git ワークフロー

- **main ブランチに直接 push しない。** 変更は必ずフィーチャーブランチを切り、Pull Request を作成する。
- 例外: `chore: bump version to ...` や `chore: release vX.Y.Z` のようなリリース関連 chore commit、および `git tag vX.Y.Z` の push は直接 main に push してよい (既存履歴もそのパターン)。
- ブランチ名は変更内容を端的に表す (例: `feat/add-only-sync-new-plugin`)。
- **PR のタイトル・本文は英語で書く。** コミットメッセージも英語。

### PR レビューサイクル

- 全 PR で **Gemini Code Assist** と **CodeRabbit** がレビューを走らせる。両 bot の投稿を待ち、コメントに対処 (fix を PR branch に push) して、フィードバックが解消してからマージする。
- **fix を push したらレビュアーに返信する。** 対応した review comment のスレッドに、**@-mention (`@gemini-code-assist` / `@coderabbitai`)** 付きで reply する。silent な fix はレビュアーから見えず、盲目的に re-review されて監査トレイル (どの fix がどの指摘に応じたか) も失われる。
- **fix + reply を送ったらそこで止まらず、能動的に bot の再発言を監視する。** 数分おき (目安 5 分程度) に `gh pr view` / `gh api .../pulls/<n>/comments` を叩いて bot の返答を確認。新しい actionable コメントが来ていれば即 fix → @-mention → 監視再開、の loop を回す。Agent 環境なら `/loop` や `ScheduleWakeup` で自動化してよい。
- **スレッド settle の判定**: 1 つの review thread は、**最新の bot 返信が ack-only** ("Thank you" / "Understood" / "Acknowledged" / 新指摘なしの re-review サマリなど) になった時点で settle。`--diff` の再指摘や追加の actionable コメントが来たら未 settle に戻す。
- **監視ストップ条件**:
  1. **すべての open thread が settle** → その PR は quiet。両 PR (or 対象 PR 全部) が quiet になった瞬間にループを抜けてオーナーに merge 判断を仰ぐ。bot が素早く ack を返した場合、30 分待つ必要はない。
  2. **bot が返信を返さないまま最後の actionable コメントから 30 分経過** → timeout としてその thread を settle 扱いにする。bot が静かに諦めるケース (actionable を止めて何も返さないモード) を拾う fallback。短すぎ (<10 分) だと遅延投稿を取りこぼし、長すぎ (>1 時間) だと merge が無意味に遅れる。
- **Merge gating.** 以下の **両方** を満たすまで merge しない:
  1. レビュー bot (Gemini / CodeRabbit) が新しい actionable コメントを出さなくなった — fix → @-mention → 沈黙、のサイクルを回し続ける。
     Bot からの "Understood" / "Thank you" のような ack のみの返信はその thread の quiet pass とみなす。新しい actionable な指摘が来たら loop を再開。
  2. リポジトリオーナー (@yukimemi) が明示的に merge を承認している。
- **例外: bot-authored PR (Renovate, Dependabot).** Gemini と CodeRabbit はデフォルトでこれらを skip するので、"bot review を待つ" gate は適用しない。CI が green で owner 承認があれば merge OK。

## Development Commands

```bash
# ビルド
cargo build

# テスト全実行
cargo test

# 単一テストの実行 (モジュール名::テスト関数名 で絞り込み)
cargo test test_generate_loader_with_cond
cargo test loader::tests
cargo test git::tests::test_git_update_method_pulls_latest

# リリースビルド
cargo build --release

# loader.lua の目視デバッグ (ignored test)
cargo test dump_full_sample_loader -- --ignored --nocapture
```

## 設計原則

**必ず TDD で実装を進める。** テストを先に書いてから (失敗することを確認して) 実装する。

**Resilience (障害耐性):** 1 つのプラグインの失敗がシステム全体を止めてはならない。同期失敗や設定ミス (依存関係の欠如など) は警告として報告し、可能な限り後続の処理 (`generate` 等) を継続する。Neovim 起動時の安全性を最優先し、不完全な設定であっても最小限の起動を保証する。

## TOML 設定スキーマ

```toml
[vars]
# ユーザー定義の変数。TOML 内 Tera テンプレートから {{ vars.xxx }} で参照できる。
repo_base   = "~/.cache/nvim/rvpm"
nvim_rc = "~/.config/nvim/rc"

[options]
# per-plugin の init/before/after.lua を置くディレクトリの root
# 未指定なら ~/.config/rvpm/<appname>/plugins
config_root = "{{ vars.nvim_rc }}/plugins"
# 並列数上限 (デフォルト 13、GitHub rate limit 回避のため控えめ)
concurrency = 16
# config.toml から外したプラグインディレクトリを sync / generate 完了時に
# 自動削除 (デフォルト false)。毎回 `sync --prune` を指定する代わり。
# auto_clean = true
# sync / generate 完了時に nvim --headless で helptags を自動生成する
# (デフォルト true)。lazy プラグインは runtimepath に載らないため、rvpm 側で
# 対象 doc/ ディレクトリを列挙して :helptags <path> を個別実行する。
# auto_helptags = false
# `rvpm add` の URL 書き込み形式: "short" (owner/repo, デフォルト) か
# "full" (https://github.com/owner/repo)。重複検出は両形式を正規化して比較。
# url_style = "full"
# rvpm のデータ置き場 root を上書き (未指定なら ~/.cache/rvpm/<appname>)。
# repos / merged / loader.lua 全部 `{cache_root}/plugins/` 配下にまとまる。
# cache_root = "~/.cache/nvim/rvpm"

[options.browse]
# README 表示を外部コマンドに委譲する (browse TUI 専用)。
# stdin に raw markdown、stdout の ANSI エスケープを ansi-to-tui 経由で
# ratatui Text に変換。失敗/タイムアウト時は tui-markdown 内蔵パスに fallback。
# placeholder は Tera 風の `{{ name }}` 記法 (rvpm 他箇所と統一):
#   {{ width }} / {{ height }} / {{ file_path }} / {{ file_dir }}
#   {{ file_name }} / {{ file_stem }} / {{ file_ext }}
# readme_command = ["mdcat"]
# readme_command = ["glow", "-s", "dark", "-w", "{{ width }}", "{{ file_path }}"]

[[plugins]]
name  = "snacks"
url   = "folke/snacks.nvim"
# on_* なし → eager (起動時にロード)

[[plugins]]
name = "telescope"
url  = "nvim-telescope/telescope.nvim"
depends = ["snacks.nvim"]
# rev: ブランチ / タグ / コミットハッシュ
# rev = "v0.1.0"

# 遅延読み込みトリガー (いずれか 1 つでも書けば lazy は自動で true に推論される)
on_cmd    = ["Telescope"]                    # string | string[]
on_ft     = ["rust", "toml"]                 # string | string[]
on_event  = ["BufReadPre", "User LazyDone"]  # "User Xxx" は User イベント + pattern に展開される
on_path   = ["*.rs", "Cargo.toml"]           # BufRead/BufNewFile の glob

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yukimemi/rvpm](https://github.com/yukimemi/rvpm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
