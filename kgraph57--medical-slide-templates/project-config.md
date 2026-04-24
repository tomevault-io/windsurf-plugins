---
trigger: always_on
description: 学会発表・抄読会・症例報告・教育講演に特化した医学スライドをHTML+CSSで生成するスキル。
---

# Medical Slide Template Skill

学会発表・抄読会・症例報告・教育講演に特化した医学スライドをHTML+CSSで生成するスキル。
国際学会（AHA/ASCO/ASH）のガイドラインに準拠したデザイン。
**slide-starterのエンジン（engine/slide.css + engine/slide.js）とテーマ（theme/anthropic.css）を直接参照する。**

トリガー: '学会スライド', '抄読会スライド', '症例報告スライド', '教育スライド', 'medical slide', 'journal club slide', 'case report slide', 'lecture slide', '発表スライド（医学）'

## ワークフロー

### Step 1: 要件を確認する

以下の必須項目をユーザーから取得する（不明なら聞く）。

| 項目 | 必須 | 例 |
|------|------|-----|
| **場面** | 必須 | A:学会口演(RCT) / B:学会口演(SR/MA) / C:学会口演(診断精度) / D:抄読会 / E:症例報告 / F:教育講演 / G:ICU症例 / H:臨床試験 |
| **テーマ** | 必須 | 小児ERにおけるAI支援トリアージの有効性 |
| **発表時間** | 必須 | 7分 / 10分 / 20分 / 30分 |
| **聴衆** | 必須 | 学会参加者 / 研修医 / 看護師 / 多職種 |
| **タイトル** | 必須 | 表紙に載せるタイトル |
| **発表者名** | 必須 | COI開示・タイトルに使用 |
| **所属** | 必須 | タイトルに使用 |

任意項目（デフォルト値あり）:

| 項目 | デフォルト | 選択肢 |
|------|-----------|--------|
| 言語 | 日本語 | 日本語 / English / バイリンガル |
| COI | なし | なし / あり（企業名を列挙） |
| 倫理審査 | あり | IRB番号・承認日を聞く |
| 枚数目安 | 発表時間×1枚/分 | 自由指定 |

### Step 2: 構成を決定する

1. 場面に応じて、場面別構成テンプレート（A-H）を選択
2. 発表時間に応じてスライド枚数を調整（1枚/分が目安）
3. ユーザーに構成案を提示し、過不足を確認
4. COI開示・倫理審査の要否を確認

### Step 3: HTMLを生成する

1. テンプレートカタログ（`decks/medical-template-catalog/index.html`）から該当テンプレートのHTML+CSSをコピー
2. コンテンツをユーザーの研究内容に差し替え
3. Chart.jsのグラフデータをユーザーのデータに差し替え
4. フッターのタイトルを更新
5. ブラウザで確認 → フィードバック → 修正のループ

### Step 4: 最終チェック

生成したスライドが以下を満たすか確認:

- [ ] COI開示スライドがタイトル直後にあるか
- [ ] 倫理審査情報が記載されているか（臨床研究の場合）
- [ ] フォントサイズが最小基準を満たしているか
- [ ] 赤-緑の色の組み合わせがないか
- [ ] 3Dグラフを使っていないか
- [ ] スライド枚数 ≦ 発表時間（分）か
- [ ] 患者個人情報（PHI）が含まれていないか
- [ ] テーブルは論文PDFのスクショでなく再タイプされているか

## CRITICAL: アーキテクチャ

anthropic-slide-styleと同じエンジン・テーマを使用する。医学特有のCSSは`<style>`に書く。

```html
<head>
  <link rel="stylesheet" href="engine/slide.css">
  <link rel="stylesheet" href="theme/anthropic.css">
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;700&family=Noto+Sans+JP:wght@400;700&family=Noto+Serif+JP:wght@900&family=Poppins:wght@400;500;600;700&display=swap" rel="stylesheet">
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    .slide h1 { font-family: "Noto Serif JP", serif; }
    /* 医学特有のCSSをここに書く */
  </style>
</head>
```

## 国際学会デザイン原則（AHA/ASCO/ASH準拠）

### 情報密度
- **1スライド = 1メッセージ**（One Slide, One Idea）
- **1スライド/分** を目安に枚数を設計
- テキスト要素は6個以下（Rule of Six）

### フォントサイズ（実装準拠）

| 要素 | サイズ | 備考 |
|------|--------|------|
| h1（タイトル） | 36-42px | slide-title のメイン見出し |
| h2（スライド見出し） | エンジンデフォルト（28-32px） | 変更しない |
| 本文テキスト | 16-22px | `.coi-box p` = 22px, `.pico-cell p` = 18px, `.sl-col li` = 18px 等 |
| テーブルセル（td） | 14-16px | 情報密度が高い要素の例外。`.table1` / `.rob-table` / `.grade-table` / `.lab-table` / `.ddx-table` = 16px |
| テーブルヘッダー（th） | 14px | 全テーブル共通 |
| ラベル/キャプション | 13-14px | `.consort-phase` = 13px, `.cc-section-label` = 13px, `.vital-label` = 14px |
| 出典/注釈 | 13px | `.muted` + `font-size:13px` |
| template-label | 11px | 開発用ラベル。発表時は非表示にする |

### カラールール
- **赤-緑の組み合わせは厳禁**（色覚多様性配慮）
- テキスト色は `#3d3d3a`（黒 `#141413` は使わない）
- アクセント色（Clay）: `#d97757`
- サブアクセント（Kraft）: `#d4a27f`
- ボーダー/ヘッダー背景（Oat）: `#e3dacc`
- ミューテッド（Muted）: `#87867f`
- **テーブルヘッダー背景**: `#e3dacc`（ベージュ）+ テキスト `#3d3d3a`
- 放射線画像は黒背景で表示
- 群間比較: Clay `#d97757` vs Muted `#87867f`（色覚安全）
- 信号色: 緑 `#059669` / 黄 `#f59e0b` / 赤 `#dc2626`
- GRADE: HIGH(緑 `#059669`) / MODERATE(黄 `#f59e0b`) / LOW(赤 `#dc2626`) / VERY LOW(紫 `#7c3aed`)

### 図表ルール
- 3Dグラフは使用禁止
- Table 1は論文PDFのスクショ貼り付け禁止 → 重要3-5項目を再タイプ
- 縦罫線は使わない（横罫線のみ）
- グリッドラインは薄く控えめに

## 場面別スライド構成テンプレート

### A: 学会口演（RCT、10分）

```
01: slide-title        — 演題名・著者・所属
02: coi-disclosure     — COI開示（必須）
03: ethics-approval    — 倫理審査情報
04: slide-section      — 背景と目的
05: slide-content      — 背景（先行研究・Clinical Question）
06: slide-landing      — 目的（1文で明確に）
07: slide-section      — 方法
08: study-design       — 研究デザイン概要（テキスト + フロー図）
09: inclusion-exclusion — 選択基準・除外基準
10: consort-flow       — CONSORT図
11: statistical-methods — 統計解析手法（cols-3カード）
12: slide-section      — 結果
13: table-1            — 患者背景テーブル
14: stat               — 主要結果の数値
15: outcome-summary    — アウトカム一覧テーブル
16: chart (bar/line)   — 主要アウトカムのグラフ
17: forest-plot / subgroup-table — サブグループ解析
18: slide-section      — 考察
19: slide-content      — 先行研究との比較
20: strengths-limitations — 強みと限界
21: slide-landing      — Take Home Message
22: acknowledgments    — 謝辞・資金源
23: references         — 参考文献
24: slide-end          — 締め
```

### B: 学会口演（SR/メタアナリシス、10分）

```
01: slide-title        — 演題名・著者・所属
02: coi-disclosure     — COI開示（必須）
03: slide-section      — 背景と目的
04: slide-content      — 背景・Clinical Question
05: slide-landing      — 目的（1文で明確に）
06: slide-section      — 方法
07: pico-table         — PICO
08: slide-content      — 検索戦略（DB, キーワード, 期間）
09: inclusion-exclusion — 選択基準・除外基準
10: prisma-flow        — PRISMA 2020 フローダイアグラム
11: statistical-methods — 統計手法（異質性評価, メタ回帰等）
12: slide-section      — 結果
13: evidence-summary   — エビデンスサマリーテーブル（複数研究比較）
14: forest-plot        — Forest Plot（主要アウトカム）
15: forest-plot        — Forest Plot（副次アウトカム/サブグループ）
16: funnel-plot        — Funnel Plot（出版バイアス評価）
17: risk-of-bias       — バイアスリスク評価
18: grade-assessment   — GRADEエビデンスプロファイル
19: slide-section      — 考察
20: slide-content      — 先行SRとの比較
21: strengths-limitations — 強みと限界
22: slide-landing      — Take Home Message
23: references         — 参考文献
24: slide-end          — 締め
```

### C: 学会口演（診断精度研究、10分）

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kgraph57) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
