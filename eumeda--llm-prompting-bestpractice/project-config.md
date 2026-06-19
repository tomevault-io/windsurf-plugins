---
trigger: always_on
description: 外部 LLM サービス (Anthropic Claude, OpenAI GPT, Edison Scientific Kosmos, Elicit, Perplexity 等) に投入するプロンプトを 3 ベンダー統合 best practice で設計・最適化する。XML タグ構造化・5 コンポーネント (role/task/context/format/constraints)・outcome-first 指示・Kosmos 用 single-objective 設計・Anti-pattern 検査を整合的に適用する。Triggers on: プロンプト最適化, prompt optimization, prompt engineering, Kosmos プロンプト, Edison プロンプト, GPT プロンプト, Claude プロンプト, XML タグ, prompt 設計, prompt review, 外部 LLM 投入, /LLM-prompting-bestpractice
---


# Skill: `/LLM-prompting-bestpractice` — 3 ベンダー統合プロンプト設計

## なぜこの skill が必要か

外部 LLM サービスに投入するプロンプト (特に Kosmos のように 12 hr 自律実行型) は、**実行中に修正できない** ため、投入前に 3 ベンダー (Anthropic / OpenAI / Edison Scientific) の公式 best practice を統合的に適用しておく必要がある。本 skill は次の 3 ステップで設計・レビューを支援する:

1. **構造化** (5 コンポーネント + XML タグ + 推奨順序)
2. **Vendor-specific 適合化** (各サービスの公式推奨と Anti-pattern)
3. **後処理 readability** (Kosmos 出力をパースする側の負荷を下げる SECTION 設計)

## When to invoke

以下のいずれかに該当する場合に invoke する:

- 外部 LLM サービス (Kosmos, Elicit, Perplexity, ChatGPT, Claude.ai 等) に投入する prompt を新規作成・レビューする
- 既存の prompt を multi-vendor で動かす (移植) ために最適化する
- ユーザーが `/LLM-prompting-bestpractice` と直接呼ぶ
- 「プロンプト最適化」「XML タグ整理」「Kosmos に投げる文章を整えて」等

- **別マシン / 自律実行用の Claude Code prompt** を作る (例: LAB-PC で長時間データ解析を 1 セッション完走させる prompt)

逆に、いま対話している Claude Code セッションへの**その場の通常 prompt** には不要。
本 skill は **(a) 外部 LLM 投入用 prompt** と **(b) 自律/別マシン実行用の agentic prompt** に適用する (後者は Template D の anti-hallucination ガードが必須)。

---

## Section 1 — 3 ベンダー共通の核原則 (6 つ)

### 原則 1: 5 コンポーネント構造 (role / task / context / format / constraints)

Anthropic Claude 4.7 は **この 5 要素パターンで訓練されており**、構造化プロンプトの方がプレーンテキストより測定可能に高品質な出力を出す ([Anthropic 2026](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices))。OpenAI GPT-5.5 も outcome-first 指示で同等の効果。

**推奨順序**:
```
1. <role>          — モデルの専門家 persona と視点
2. <context>       — フィールド固有の前提・データ来歴・なぜ重要か
3. <task>          — 単一の主目的 (research objective)
4. <instructions>  — 番号付き手順 (sequential steps)
5. <output_format> — SECTION 構造 + 制約
```

**理由**: context を task より先に置くと、Claude は task をどの枠組みで解釈すべきかを先に理解する。これは long-context (20k+ token) で 30% 精度改善 ([Anthropic Long context tips](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices))。

### 原則 2: XML タグでセクション分離

3 ベンダー共通で XML タグは parsing を unambiguous にする。Markdown header (`## `) より強い。

**標準タグ語彙 (Anthropic 公式)**:
- `<role>`, `<context>`, `<task>`, `<instructions>`, `<output_format>` — 主構造
- `<example>`, `<examples>` — 多ショット用 (3-5 件で best)
- `<documents>`, `<document index="n">`, `<document_content>`, `<source>` — 複数ファイル投入時
- `<constraint>`, `<scope_boundary>`, `<do_not>` — 禁則事項
- `<thinking>`, `<answer>` — chain-of-thought 構造化

**ネスト**: 階層がある場合は必ずネスト。例:
```xml
<documents>
  <document index="1">
    <source>dataset.csv</source>
    <document_content>...</document_content>
  </document>
</documents>
```

**注意**: タグ名は識別性が大事 (canonical name はない)。データ内容と意味が一致する名前を選ぶ。

### 原則 3: Outcome-first (path ではなく target を指定)

GPT-5.5 (2026-04) と Claude Opus 4.7 (2026-05) はいずれも **「to-do list」より「success criteria」を好む** ([OpenAI GPT-5.5 guide](https://cookbook.openai.com/examples/gpt-5/gpt-5_prompting_guide))。

**Bad (path-prescriptive)**:
```
Step 1: Read the PDF.
Step 2: Extract all species names.
Step 3: Search PubMed for each species.
Step 4: Compile results into a table.
```

**Good (outcome-first)**:
```
Produce an extension table containing records not present in the canonical
database. Each new record must include source DOI, observation year, location,
and a reliability rank A/B/C. Success criterion: no record duplicates an
existing canonical entry (or a known synonym of one).
```

**理由**: モデルの reasoning engine は path を自分で最適化する。過剰指示は探索空間を狭める。

### 原則 4: 絶対ルールは sparingly に (true invariants のみ)

OpenAI GPT-5.5 (2026-04) の重要な方針転換: **「MUST/NEVER は乱発しない」**。Claude 4.7 も literal interpretation で同様の挙動。

**MUST/NEVER を使ってよいケース** (true invariants):
- safety / 倫理規定
- 必須出力フィールド
- 投入禁止データ (例: 未発表 simulation 結果)
- スコープ境界 (out-of-scope の明示)

**MUST を使ってはいけないケース** (= soft preference):
- スタイル指示 (例: "MUST use academic tone" → 単に "Write in academic tone")
- 望ましい構造 (例: "MUST use bullet lists" → "Prefer bullet lists")
- 推奨される手順 (例: "MUST search PubMed first" → "Start by searching PubMed")

濫用は逆効果 (モデルが本物の constraint を見分けられなくなる)。

### 原則 5: 否定より肯定指示

3 ベンダー共通: **「X しないで」より「Y して」**。

**Bad**: "Do not use markdown."
**Good**: "Respond in flowing prose paragraphs."

**Bad**: "Don't be vague."
**Good**: "State the species name, location, year, and reliability rank for each record."

例外: out-of-scope 明示 (これは肯定的「禁止」として有効)。

### 原則 6: canonical examples を必ず同梱 (multishot) — 最重要

Anthropic 公式が**最も信頼できる steering 手段**と明言: 「Examples are one of the most reliable ways to steer Claude's output」「3–5 個推奨」「examples are the 'pictures' worth a thousand words」([Anthropic 2026](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices); [context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents))。ルールを並べるより**例を示す**方が format・tone・判定を確実に伝える。

- **3–5 個の canonical example** を `<example>` (複数なら `<examples>`) で囲んで同梱する。
- 例は **relevant** (実 use case を写す)・**diverse** (edge case を含み、意図しないパターンを拾わせない)・**structured** (タグで指示と分離)。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eUmeda/LLM-prompting-bestpractice](https://github.com/eUmeda/LLM-prompting-bestpractice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
