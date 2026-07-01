---
trigger: always_on
description: このリポジトリは TC39 plenary の議事録(`raw/notes`)を素材に、**「各提案がどういう経緯でステージを変えてきたか」「策定途中で何が論点になったか」を後から辿れる wiki** を LLM が構築・維持するためのものです。設計思想は [llm-wiki.md](llm-wiki.md) を参照。
---

# TC39 Wiki — Agent Schema

このリポジトリは TC39 plenary の議事録(`raw/notes`)を素材に、**「各提案がどういう経緯でステージを変えてきたか」「策定途中で何が論点になったか」を後から辿れる wiki** を LLM が構築・維持するためのものです。設計思想は [llm-wiki.md](llm-wiki.md) を参照。

このファイルは wiki の運用規約(schema)です。新しいセッションはまずこれを読み、規約に従って ingest / query / lint を行ってください。

## レイヤ

- **Raw sources** — `raw/`(submodule 群)。**読み取り専用・不変**。絶対に編集しない。これが真実の素材。
  - `raw/notes/`(tc39/notes) — plenary の逐語録。**経緯・論点・発言の一次ソース**。
  - `raw/proposals/`(tc39/proposals) — 提案の正典リスト(現ステージ別テーブル + champions)。**現ステージ・status・champion の確定値の一次ソース**。
- **The wiki** — `wiki/`。LLM が全面的に所有。提案ページ・index・log を生成/更新する。
- **Generated** — `wiki/_generated/`、`wiki/people/`、`wiki/proposals/index.md`。`tools/` のスクリプトによる機械抽出物。**手で編集しない**(再生成で上書きされる)。

**ソース間の優先順位(precedence)**: 食い違ったときにどれを信じるか。

- 経緯(ステージ遷移の年月・方向)・論点・発言の帰属 → **`raw/notes`** が一次。
- 提案の**現ステージ(`current_stage` / `status`)と champion** の確定値 → **`raw/proposals`** が一次。notes から推論した現ステージが proposals と食い違ったら、まず notes 側の読み取りを疑い、両者を辿り直して解決する。
- `raw/proposals` は**現在のスナップショット**であり経緯は持たない。逆に notes は経緯を語るが現ステージの確定には弱い。役割が異なるので、片方だけで埋めず両者を突き合わせる。

## ディレクトリ構成

```
raw/notes/meetings/<YYYY-MM>/<month-DD>.md   素材(逐語録)
raw/proposals/README.md                      Active 提案テーブル(Stage 3 / 2.7 / 2)
raw/proposals/finished-proposals.md          Stage 4(出荷済み)
raw/proposals/stage-1-proposals.md           Stage 1
raw/proposals/inactive-proposals.md          withdrawn / inactive
wiki/
  README.md                 提案カタログ(現ステージ付き、カテゴリ別)
  log.md                    時系列の ingest/query/lint ログ(append-only)
  proposals/<slug>.md       提案ごとの精読ページ(経緯+論点)
  proposals/index.md        全提案のステージ一覧(生成物。Stage 4 は未収載分のみ。raw/proposals から生成。手で編集しない)
  families/<family>.md      カテゴリ横断のまとめ(複数提案を束ねる synthesis。個別の経緯は proposals を参照)
  meetings/<YYYY-MM>/        会合の日次要約(summarise の出力。日ごと 1 ファイル + README.md)
  people/<ABBR>.md          人物リファレンス(生成物。filename = 略号)
  _generated/
    agenda-index.md         全86会合の議題インデックス(grep 用バックボーン)
    agenda-index.jsonl      同上の機械可読版
tools/
  extract_agenda.py         agenda-index の生成スクリプト
  extract_proposals.py      raw/proposals から全提案ステージ一覧 wiki/proposals/index.md を生成(Stage 4 は未収載分のみに filter)
  extract_people.py         提案ページに登場する人物の people/ ページ生成
  link_people.py            提案ページ中の略号を [ABBR](../people/ABBR.md) にリンク
```

## 言語規約

- 地の文(概要・経緯・論点の説明)は **日本語**。
- **提案名・Stage 表記・API 名・spec 用語・人物の略号(略号は原文のまま)は英語**。例: `Temporal`, `Stage 2.7`, `Array.fromAsync`, `[[Get]]`, `PFC`。
- **議事録から発言(セリフ)を引用するときは日本語に翻訳する**。原文が英文一文以上のときは訳文を載せる(例: WH「変えないでほしい」)。単語・短い専門句(`muddled`, `uninitialized function` 等)は英語のままでよい。提案名やアジェンダ項目名のような固有のタイトルは原文のまま。

## 提案ページの形式

`wiki/proposals/<slug>.md`。`<slug>` は kebab-case の英語(例: `temporal`, `decorators`, `records-and-tuples`)。

先頭に YAML frontmatter(Obsidian Dataview 用):

```yaml
---
title: Temporal
slug: temporal
status: shipped        # stage0 | stage1 | stage2 | stage2.7 | stage3 | shipped | withdrawn | inactive
current_stage: 4       # 0 / 1 / 2 / 2.7 / 3 / 4
ecma: [262, 402]       # 影響する仕様
champions: [PFC, ...]  # 略号
first_seen: "2017-09"  # 初出会合(YYYY-MM)
reached_stage4: "2026-03"
families: [date-time]  # 所属する family(任意・複数可。`wiki/families/<family>.md` と双方向に対応)
tags: [proposal, date-time]
---
```

本文セクション(見出しは固定):

1. `## 概要` — 1〜3 段落。何を解決する提案か。
2. `## ステージ遷移` — 時系列テーブル。1 行 = 1 イベント:

   | 会合                                     | できごと                             | Stage |
   | ---------------------------------------- | ------------------------------------ | ----- |
   | [2018-09](../_generated/agenda-index.md) | Stage 2 到達。`Temporal for Stage 2` | 1 → 2 |

   会合セルは `raw/notes` の該当ファイルへ相対リンク(例: `[2018-09](../../raw/notes/meetings/2018-09/sept-27.md)`)。Stage 列は遷移を `旧 → 新`、更新のみなら現ステージを記す。テーブルの直後に、下記のステージ推移グラフを置く。

3. ステージ推移グラフ — テーブルの下に mermaid `xychart-beta` の折れ線を埋め込む。**横軸は議事録のある全区間(2012〜2026 の年)固定**、縦軸は Stage (0〜4)。各年末時点の stage を下から積み上げる形で並べる。提案が存在しない年は 0。Stage 2.7 を経た提案は `2.7` を小数点で打つ。**撤回された提案は撤回年で線を止める**(line 配列をそこで終え、以降の点を描かない)。長期停滞は同じ値の横ばいで自然に表現される(特別な印は不要)。グラフ直後に読み方の注記(各遷移の年月)を `>` で添える。例:

   ````
   ```mermaid
   xychart-beta
       title "Temporal stage 2012-2026"
       x-axis [2012, 2013, 2014, 2015, 2016, 2017, 2018, 2019, 2020, 2021, 2022, 2023, 2024, 2025, 2026]
       y-axis "Stage" 0 --> 4
       line [0, 0, 0, 0, 0, 1, 2, 2, 2, 3, 3, 3, 3, 3, 4]
   ```
   ````

   注: `xychart-beta` は mermaid 10.3+ が必要。VSCode の `bierner.markdown-mermaid` の webview プレビューでは空描画になる(別の mermaid 拡張なら描画可)ため、レンダラ依存に注意。title は ASCII 推奨(全角・em ダッシュ・括弧で parse が崩れる環境がある)。

4. `## 主な論点` — 策定途中で問題になった点。論点ごとに小見出し `### <論点名>`。各論点に: 何が争点か / 誰が懸念したか(略号) / どの会合で / どう決着したか(または未決)。発言引用は `>` で(日本語訳)。
5. `## 関連提案` — `[Title](../proposals/other-slug.md)` 形式で相互リンク(未作成提案はコード表記の素テキスト)。
6. `## 出典` — 参照した会合ファイルの一覧(箇条書きリンク)。

リンク規約: **すべて標準の markdown 相対リンク**を使う(Obsidian の `[[wikilink]]` は VSCode の markdown プレビューで遷移できないため使わない。標準リンクは VSCode でも Obsidian でも動く)。

- 素材へ: `[2018-09](../../raw/notes/meetings/2018-09/sept-27.md)`
- 提案間: `[Temporal](../proposals/temporal.md)`。まだ作成していない提案はデッドリンクを避け、コード表記の素テキスト(例: `` `pattern-matching` ``)で書き、作成時にリンク化する。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jxck/tc39-llm-wiki](https://github.com/Jxck/tc39-llm-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
