---
trigger: always_on
description: narou.rb（Ruby製の日本のWeb小説管理・電子書籍変換ソフトウェア）のサーバー実行部分をRustに移植するプロジェクト。なろう・カクヨム等のサイトからのDL・変換が動作し、narou.rbの出力フォーマットと完全互換性を持つことを目指す。
---

# narou.rs — Rust Port of narou.rb

## Overview
narou.rb（Ruby製の日本のWeb小説管理・電子書籍変換ソフトウェア）のサーバー実行部分をRustに移植するプロジェクト。なろう・カクヨム等のサイトからのDL・変換が動作し、narou.rbの出力フォーマットと完全互換性を持つことを目指す。

## 実装状況
`COMMANDS.md` が narou.rb 全24コマンドのオプション・挙動とRust側実装状況を管理するマスタードキュメントである。最新の実装状況はそこを参照すること。

| 完了度 | コマンド数 | 内訳 |
|:------:|:---------:|------|
| ✅ 完了 | 18 | init, list, tag, freeze, remove, setting, diff, send, backup, clean, help, version, log, folder, browser, alias, inspect, csv, trace |
| 🟡 部分 | 4 | download, update, convert, web |
| 🟡 部分 | 1 | mail |
| ❌ 未実装 | 1 | — (全コマンド実装済み) |

## Porting Policy
- このプログラムは `sample/narou` にある本家 narou.rb を Rust へ移行するための互換実装である。
- 内部ライブラリ、データ構造、処理系統、実装アルゴリズムは Ruby 版と同一である必要はない。Rust 側で保守しやすく、安全で、検証しやすい構成を優先してよい。
- 互換性の主対象は外部から観測できる挙動である。特に CLI/API の引数・戻り値・エラー挙動、`webnovel/*.yaml` や `converter.yaml` などの YAML 構文理解、`.narou/` 配下のデータ読み書き、最終的なファイル出力を narou.rb と徹底的に合わせる。
- Ruby 実装は仕様の参照元として扱う。処理手順をそのまま写すことよりも、同じ入力から同じ外部挙動・同じ出力を得ることを優先する。
- 互換性調査では Ruby 版の内部手順を読むが、それは外部仕様を抽出するためである。Rust 実装では、外部挙動・データ互換・出力互換を壊さない限り、Ruby の逐語的移植よりも堅牢性、保守性、検証容易性、性能、安全性が高い設計を選ぶ。

## 互換性の要件レベル
- 外部から観測できる挙動の互換性は**妥協せず完璧に**追求する。これには以下が含まれる:
  - **設定ファイルの位置**: `.narou/local_setting.yaml`、`~/.narousetting/global_setting.yaml` など、Ruby 版と同一パスに配置する。
  - **設定ファイルの読み書き互換**: Rust が書いた YAML を Ruby が読め、Ruby が書いた YAML を Rust が読めること。`---` ヘッダの有無など形式の差は許容されるが、意味論（キー名・値の型・構造）は一致させる。
  - **全設定項目の読み書き**: Rust 側に未実装の機能（send、mail、device 変更自動調整等）の設定項目であっても、`narou setting` コマンドで読み取り・設定・削除が可能であること。`default.*`、`force.*`、`default_args.*` 系の動的変数名もすべて受け付けること。
  - **CLI の引数・戻り値・エラーメッセージ・終了コード**: Ruby 版と同一であること。
  - **`webnovel/*.yaml` や `.narou/` 配下のデータ構造**: Ruby 版が読める形式を維持すること。
  - **最終的な変換出力ファイル**: narou.rb の出力と同一であること。
- 「内部実装は異なってよい」方針は変更しない。上記の外部互換性を満たす限り、Rust 側のアルゴリズム・データ構造・処理順序は自由に選んでよい。Ruby 版に既知の脆さや古い都合がある場合は、同じ外部結果になることをテストやドキュメントで確認した上で、Rust 側ではより良い内部設計を採用する。

## YAML-Driven Site Definition Compatibility
- サイト別の取得・前処理・抽出ルールは narou.rb と同じく `webnovel/*.yaml` を主たる仕様として扱う。ユーザーが初期化フォルダ内の `webnovel/*.yaml` を編集・差し替えた場合、その内容で挙動を変えられることが互換性の重要要件である。
- Rust 側にサイト固有ロジックを直接ハードコードする実装は、最終的な互換方針としては不可。特に `code: eval:` や前処理相当の記述を YAML から切り離して Rust 関数へ固定すると、narou.rb の「YAML を更新すればサイト追従できる」という性質を壊す。
- 2026-05 時点: ハードコードされた `kakuyomu_preprocess` は完全に除去され、`webnovel/kakuyomu.jp.yaml` の `preprocess:` DSL ブロックへ移行済み。pest 文法ベースの安全な DSL パーサー (`src/downloader/preprocess.pest`) + インタプリタ (`src/downloader/preprocess/interpreter.rs`) により、YAML 記述だけでカクヨム JSON → 中間テキストの展開が可能である。ユーザー側 YAML の `preprocess:` を編集するだけで前処理ロジックを差し替えられる。
- pest 文法 (`src/downloader/preprocess.pest`) は以下の構文に対応: `guard`/`let`/`set`/`if`/`else`/`for`/`emit`/`insert_at_match`, 文字列補間 `${...}`, 正規表現 JSON 抽出 `extract_json(/.../)`, メソッドチェイン `.map`/`.flat_map`/`.flatten`/`.compact`/`.join`/`.gsub`/`.replace`/`.is_array`/`.empty`, 論理演算 `&&`/`||`/`!`/`==`/`!=`。実行時に step budget / 文字列サイズ上限 / 配列要素数上限による防御あり。
- 新しいサイト対応やサイト構造変更対応では、まず YAML 表現で解決できるかを検討する。やむを得ず Rust に暫定処理を置く場合は、暫定であること、対応する YAML 意味論、将来 YAML 駆動へ戻す作業を `AGENTS.md` または Serena メモに明記する。
- Arcadia (`webnovel/www.mai-net.net.yaml`) に `encoding: UTF-8` は置かない。narou.rb の同梱 Arcadia 定義には無く、Rust 側は UTF-8 を既定として扱えばよい。Arcadia の本文取得不具合の実原因は `href` の `&amp;` を未デコードのまま section URL に使っていたことであり、`build_section_url()` 側で HTML エンティティを復元する。

## COMMANDS.md 同期ルール
- `COMMANDS.md` は narou.rb 全24コマンドのオプション・挙動と Rust 側実装状況を管理するマスタードキュメントである。
- **コマンドの新規実装・オプション追加・フラグ追加・挙動変更を行うたびに、必ず `COMMANDS.md` の該当箇所をリアルタイムに更新する。**
- 更新内容: Rust 列の ✅/🟡/❌ マーク、実装状況サマリの完了度、不足動作リストの削除・追加。
- 実装が完了したコマンドは「部分」→「完了」に昇格させる。
- 全24コマンドが narou.rb と完全互換になるまで、この同期作業を継続する。
- Serena メモリにも常に最新の実装状況を反映する。
- **完了判定の注意**: `COMMANDS.md` の ✅ 完了は、Rust 側に該当処理や help 表示が存在するだけでは付けない。必ず Ruby 版 `sample/narou/lib/command/*.rb` と、CLI オプション、help 文、Examples、設定項目、終了コード、エラー文、未実装の周辺動作を細かく突き合わせ、外部から観測できる挙動が一致していることを確認してから完了にする。
- 特に `help` は未実装コマンド分も narou.rb から移植する方針のため、Rust 側の実装済みコマンド集合と比較して完了判定しない。`narou <command> -h` の詳細文、Options、Configuration、Variable List、Examples を Ruby 版の各 command ファイルと比較して判断する。
- 既に ✅ と書かれているコマンドでも、同じ節に「未実装」「不足動作」が残っている場合や Ruby 版 help/挙動との差分がある場合は、実態に合わせて 🟡 部分へ戻す。完了度は楽観的に維持せず、互換性確認の粒度を優先する。

## コミット時のコード整形禁止ルール
- git diff に現れる変更は、機能的な意味を持つものだけにすること。
- コードの見た目だけを変える無意味な変更を禁止する。具体的には以下:
  - 既存の一行を複数行に改行+インデントし直すだけの変更
  - 既存の複数行を一行にまとめ直すだけの変更
  - `use` / `import` の順番を入れ替えるだけの変更
- これらの整形変更は、機能変更に付随して不可避な場合（例: 引数追加で行長が変わる）のみ許容する。

## Git 運用ルール
- 通常の修正・軽微な機能追加・ドキュメント更新は `develop` 上で行う。作業開始前に現在ブランチと作業ツリーを確認し、`main` 上で直接作業しない。
- 作業開始時に対象ブランチが `origin` より遅れている場合は、`git pull` で最新へ追従してから作業を始める。
- 大幅な変更、新機能、複数ファイルにまたがる設計変更、長時間かかる検証を伴う作業は、`develop` から機能単位のブランチを作成して進める。
- 機能ブランチ名は内容が分かる短い英数字・ハイフン形式にする。例: `fix-web-concurrency`, `feature-series-url`。
- 機能ブランチでは適切な動作テストを済ませてから `develop` に統合する。統合後も `develop` 上で必要なテストを再実行する。
- `main` への統合は、ユーザーが明示的に依頼した場合、またはリリース作業として明確に合意された場合だけ行う。`develop` は削除せず残す。
- `main` へ統合する前に、`develop` が clean であること、必要なテストが通っていること、バージョン更新や README 更新などリリースに必要な差分が揃っていることを確認する。
- タグ作成はユーザーがバージョン番号を明示した場合だけ行う。タグは `main` のリリースコミットを指すようにし、作成後に push する。
- 実装が一区切りついたら、機能単位で git commit する。無関係な変更をひとつの commit に混ぜず、レビューやロールバックがしやすい粒度に分ける。
- commit 前には `git diff` / `git status` を確認し、ユーザー由来または別作業由来の変更を混ぜない。意図しない整形差分、改行だけの変更、import 並び替えだけの変更を含めない。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rumia-Channel/narou.rs](https://github.com/Rumia-Channel/narou.rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
