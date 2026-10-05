---
trigger: always_on
description: このリポジトリで作業する AI エージェント (と人間の開発者) 向けの案内。利用者向けの説明は [README.md](README.md) にある (日本語だけで書く。英語版は置かない)。
---

# noslop — エージェント向けガイド

このリポジトリで作業する AI エージェント (と人間の開発者) 向けの案内。利用者向けの説明は [README.md](README.md) にある (日本語だけで書く。英語版は置かない)。

## プロジェクト概要

noslop は、日本語の文章から「AI 臭さ」を機械的に拾う Rust 製の Linter。判定器ではなく、疑わしい箇所を決定的に並べ、直すかどうかは書き手に委ねる。

- 文字種・語句パターン・文長の統計で判定する。品詞で数えるルール (P15・P16・R18・R20) は hasami の辞書で判定する (`morph.rs`)。既定 (`dictionary = "auto"`) では、share ディレクトリに取得した配布辞書を hasami の推奨順で選び、なければバイナリに同梱した IPAdic の辞書 (`dict/ipadic.hsd`) を使う。元の校正は同梱の IPAdic で行ったので、その条件で判定するなら `dictionary = "bundled"`。`--no-dict` や、同梱しないビルドで辞書が見つからないときは辞書なしの近似で動く
- 設定はユーザーの設定 (`~/.config/noslop/config.toml`) とプロジェクトの設定 (`noslop.toml`) の 2 層で、既定値 < ユーザー < プロジェクト < CLI の順に項目ごとに重ねる (`config.rs`)
- 文書 (Markdown・テキスト) のほかに、コードのコメント・静的な文字列・JSX/HTML の本文と表示属性 (tree-sitter で取り出す。`code/`) を検査できる。コメントや表計算のセルのような互いに独立した短い断片の集まり (`DocumentKind::Fragments`) には、1 文ずつ判定するルール (`RuleUnit::Sentence`) だけを当てる。文書・段落をまたいで数えるルールは、ひと続きの地の文で校正したため
- 既定で有効にするのは、コーパスで誤検知率を確かめた語句と閾値だけ。未校正のものは experimental にする
- AI 臭さ (`slop`) と読みやすさ (`readability`) の 2 つのレーンを混ぜない
- 文書全体の点数は出さない。指摘を 1 件ずつ並べ、レーンと重大度ごとの件数を数えるだけにする (以前の「自然度スコア」は校正しておらず、人の文書と AI の文書を見分けていなかったので外した。[点数を出さない理由](docs/revision.md#文書全体の点数を出さない理由))

## 技術スタック

- Rust (edition 2024)。ツールチェーンの版は `mise.toml` が正
- Markdown: `pulldown-cmark`
- 文分割と形態素解析: [hasami](https://github.com/owayo/hasami)。文分割は `hasami::sentence` (辞書を使わない)、形態素解析は `analyzer` feature (辞書があるときだけ)。crates.io の同名クレートは別物なので、git の依存でリリースのタグ (`Cargo.toml` の `tag`) を指定して入れる。テストは `build` feature (dev-dependency) で小さな辞書を組み立てる。既定の feature `bundled-dict` で `dict/ipadic.hsd` を `hasami::include_hsd!` でバイナリに埋め込み、`Dictionary::from_static` で複製せずに読む。辞書を指定しないとき (`auto`) は、share ディレクトリに取得した配布辞書を同梱の辞書より先に使う (選ぶ順は依存の hasami の推奨順で、目録の自動更新では変わらない)。辞書を差し替える手順は `dict/README.md`。上げるときはタグを書き換えて `cargo update -p hasami` を実行し (hasami の `rust-version` が上がっていれば、`Cargo.toml` の `rust-version` もそろえる。上げると clippy が MSRV で抑えていた指摘を出すことがある)、例外表の版を固定したテスト (`segment.rs`) が落ちたら分割の差分と THIRD_PARTY_NOTICES.md の NOTICE の写しを確かめる。依存の更新で HSD の形式が変わるときは、配布辞書の目録 (`dict/catalog.json`) と同梱の辞書も同じ形式に更新する。目録の辞書の形式 (`format_version`) が依存の `hasami::hsd::FORMAT_VERSION` と同じことはテスト (`the_catalog_is_readable_by_the_linked_hasami`) が確かめる。同梱の辞書を上げる手順は `dict/README.md` にある (テスト `the_bundled_dictionary_matches_dict_readme` が `dict/ipadic.hsd` を `dict/README.md` の表の SHA-256 と照らす)
- 配布辞書の目録: `dict/catalog.json` (hasami のリリースに添付された `dictionaries.json` をそのまま置く) が正本。`build.rs` が serde・serde_json (build-dependencies) で読んで検証し、`HASAMI_TAG`・`DEFAULT_SOURCE`・`CATALOG_FORMAT_VERSION`・`RECOMMENDED`・`DICTIONARIES` を作る。`src/dictionaries.rs` はそれを `include!` で取り込む (壊れた目録ではビルドが止まる)。目録は Release のワークフローが hasami の最新のリリースに合わせて自動で更新する。手で更新するなら `make dict-catalog` (`TAG=...` で版を指定、省くと最新。`tools/dict-catalog.sh` が gh で取得し、古い版には戻さない)、目録の辞書を実際に取得できるかは `make dict-check` (通信が要る) で確かめる。依存の hasami (`Cargo.toml` のタグ) と同梱の辞書は判定の結果を左右するので、目録とは別に人が上げる
- コードのコメントの取り出しと、gws のコマンドの解析: `tree-sitter` (0.27) と言語ごとの文法の crate。文法はすべて常にバイナリに入れる (feature で分けない。どの配布物でも同じ言語を読めるように。バイナリの大きさは、同梱辞書とターゲットによって変わる)。Bash の文法は gws のコマンドの解析にも使う。Ruby の文法は astro-sight と同じく owayo のフォーク (git の依存) を使う。SQL の文法 (tree-sitter-sequel) は `cc` を `~1.2` に固定させ、`ring` が引く新しい `cc` とぶつかるので入れない。文法の crate は tree-sitter 0.27 で読める版に固定している (上げるときは文法ごとに ABI の対応を確かめる。0.27 の API の変更は tree-sitter-guide スキル)。文法は配布のバイナリに静的にリンクするので、THIRD_PARTY_NOTICES.md の「tree-sitter と言語の文法」の表に版と著作権の表示を載せている。文法を足す・上げるときは表も直す
- 語句の照合: `aho-corasick` / `regex`
- ファイル探索: `ignore` (.gitignore を尊重)、並列化: `rayon`
- CLI: `clap`、設定: `toml` + `serde`、出力の色: `anstream` / `anstyle`
- 配布辞書の取得 (`noslop dict download`): hasami の feature `download` (`hasami::download`) に任せる。HTTP は hasami が引く `ureq` (TLS は rustls。社内の CA を入れた環境でも通るよう、OS の証明書ストアで検証する `platform-verifier`)、圧縮版 (`.hsd.zst`) の展開は `ruzstd`、検証は `sha2`、置き換えは `tempfile`。noslop は目録の値と取得元を渡し、誤りを日本語に言い換える。取得の細部 (受信・展開・照合・一時ファイル) のテストは hasami にあり、noslop は目録の渡し方・言い換え・CLI を確かめる

## 構成

```mermaid
flowchart TD
    CLI[cli.rs<br/>引数・サブコマンド] --> CFG[config.rs<br/>ユーザーの設定と noslop.toml の読み込み・重ね合わせ]
    CLI --> CHANGED[changed.rs<br/>git の差分・変更行との重なり]
    HOOK --> CHANGED
    CLI --> WALK[walk.rs<br/>対象ファイルの列挙]
    WALK --> DOC[document.rs<br/>Document / Block / Sentence]
    DOC --> MD[markdown.rs<br/>Markdown → ブロック]
    DOC --> TXT[plaintext.rs<br/>テキスト → ブロック]
    DOC --> CODE[code/<br/>コードの文言 → ブロック]
    MD --> DIR[directive.rs<br/>抑制コメントの読み取り]
    TXT --> DIR
    DOC --> SEG[segment.rs<br/>文分割]
    CFG --> ENG[engine.rs<br/>ルールの選択・実行]
    DOC --> ENG
    ENG --> RULES[rules/<br/>phrases・rhythm・structure・custom]
    ENG --> MORPH[morph.rs<br/>形態素解析 (辞書は同梱)]
    RULES --> MORPH
    ENG --> SUP[suppress.rs<br/>抑制の適用]
    ENG --> OUT[output/<br/>text・json・toon・github・brief]
    CLI --> DIFF[diff/<br/>改稿の前後の比較]
    CLI --> CAL[calibrate.rs<br/>コーパスでの校正]
    CLI --> MCP[mcp.rs<br/>MCP サーバー]
    CLI --> HOOK[hook/<br/>Claude Code・claw-hooks のフック]
    HOOK --> GWS[gws/<br/>gws で書き込む値の取り出し]
    CLI --> DICTS[dictionaries.rs<br/>配布辞書の取得]
    MORPH --> DICTS
    DIFF --> ENG
    CAL --> ENG
    MCP --> ENG
    HOOK --> ENG
    MCP --> OUT
    HOOK --> OUT
```

## モジュールの責務

| ファイル | 責務 |
|---------|------|
| `src/main.rs` | エントリポイント。終了コードを返す |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [owayo/noslop](https://github.com/owayo/noslop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
