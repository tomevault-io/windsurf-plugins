---
trigger: always_on
description: >-
---


# Tutorial authoring

Use this skill when writing, revising, or auditing any document
where a reader follows steps to build or achieve something.

## Target learner (expertise reversal boundary)

This skill is optimised for **beginner-to-intermediate**
learners encountering the subject for the first or second time.
Most multimedia learning principles (Signaling, Pre-training,
Personalization, heavy imagery) are strongest in that range and
weaken — or reverse — for experts (Kalyuga's *expertise
reversal effect*). If the artefact is an expert-facing quick
reference, the author MUST scale back Signaling, Concept
density, and hand-holding narrative, and lean on Reference
tables. When in doubt, state the target learner explicitly in
the document's opening.

## Scientific foundations

All authoring rules below derive from the principles in this
table. Principles are stated so their scopes do NOT overlap;
when two seem to conflict, the "Scope & limits" column
resolves the boundary. The agent MUST apply them actively when
writing new tutorials and when reviewing existing ones.

### Underlying load model (Sweller's CLT)

Every principle in the table below is a tactic for managing one
of the three load types in Cognitive Load Theory (Sweller, 1988).
When two principles compete, resolve by asking *which load type
currently dominates*.

| Load type | What it is | Which principles address it |
|---|---|---|
| **Intrinsic** — inherent difficulty of the material | Cannot be reduced, only sequenced | Segmenting, Pre-training, Activation |
| **Extraneous** — effort wasted on poor presentation | MUST be minimised | Coherence, Redundancy, Spatial/Temporal contiguity, Split-attention, Signaling, Modality |
| **Germane** — effort spent on schema construction | SHOULD be fostered | Multimedia, Personalization, Generative activity, Worked example, Feedback |

Expertise reversal (Kalyuga, 2007) predicts that tactics which
reduce extraneous load for novices can *increase* extraneous
load for experts (because redundant signals compete with
established schemas). This is why the skill scopes itself to
beginner-to-intermediate learners.

| Principle (source) | Core insight | Scope & limits | Authoring implication |
|---|---|---|---|
| **マルチメディアの原理** (Mayer, 2009) | 補完的情報を異なる表現（画像とテキスト）に分担すると学習が促進される | 「組み合わせ」は**補完**であって**重複**ではない。同一情報の二重提示は本原理では正当化されない(→ 冗長性原理) | 操作ステップでは、画像が WHERE（位置・順序・選択肢の外観）を、テキストが WHAT（動作の種類・画像に映らない値）を担う |
| **空間的接近の原理** (Mayer, 2009) | 対応する画像とテキストが空間的に近いほど効果的 | 1:1 対応のペアに限定。無関係な画像とテキストを並置する理由にはならない | 1 Action = 1 画像。画像は対応テキストの直前・隣接に配置する |
| **時間的近接の原理** (Mayer, 2009) | 対応する画像とテキスト（または音声）は同時に提示するほど効果的 | **音声または動画など時間軸を持つ媒体にのみ**適用。静的ページでは空間的接近原理で代替 | ナレーション付き動画では、画像切替とナレーションを同期させる |
| **一貫性の原理** (Mayer, 2009) | 教示目的と無関係な文書・画像・音は学習を阻害する | 「無関係」は学習目的から見た判定。面白さや装飾性は保持の根拠にならない | 装飾画像・余談・BGM・装飾的アニメーションは除去 |
| **モダリティの原理** (Mayer, 2009) | 視覚＋聴覚の分担は視覚独占より有効（視覚チャネル過負荷回避） | **音声モダリティを含む媒体（動画・音声教材）でのみ**適用。静的テキスト＋画像の媒体では無関係 | ナレーションと同一文章を画面に出さない |
| **冗長性の原理** (Mayer, 2009) | 意味的に同一の情報を複数フォーマットで重複提示すると学習を阻害する | 適用対象は**意味的に同一**の情報（同じ UI ラベル・同じ値・同じ説明）に限定。補完的情報の併置は該当しない(→ マルチメディア原理) | 画像に映っている UI ラベル・選択肢名・既定値をテキストで再掲しない |
| **セグメンティングの原理** (Mayer, 2009) | 学習者がペースを制御できる単位に分割するほど効果的 | セグメント単位は**1つの意味的に閉じたサブゴール**。単一画面・単一状態内の連続操作は原則 1 セグメント。画面遷移・状態遷移・モード切替が自然な境界 | 画面内の項目数で機械的にセグメントを割らない。画面遷移で区切る |
| **分割注意の原理** (Ayres & Sweller, 2021) | 空間的に離れた複数の情報源を統合する必要があると外在的処理が増大する | スクリーンショットと注釈テキストの**物理的距離**が問題。空間的接近原理と相補関係にあるが、こちらは**離れた情報源の統合コスト**に焦点を当てる | 番号吹き出し付きスクリーンショットと説明テキストを隣接配置する。ページ下部にまとめた「設定一覧表」から遠いスクリーンショットを参照させる構成を避ける |
| **ミニマリズム P1: 行動志向** (van der Meij & Carroll, 1995; Carroll, 1990) | 学習者はすぐ行動しながら学ぶ（doing で学ぶ）。最初のアクションへの到達を最短にする | 適用対象は**まだ不要な情報の除去**。必要情報の補完的分担(→ マルチメディア原理)を削ることは含意しない | 前置きの概念説明を最小化し、最初の Action を早める。Concept は first-use 直前に置く |
| **ミニマリズム P2: タスク領域への定着** (van der Meij & Carroll, 1995) | 教材は学習者の実際の目標とタスクに基づく。機能ベースではなく目標ベースで構成する | 「実タスク」は学習者が**本当に達成したいこと**を指す。ソフトウェアの機能一覧に沿った構成はこの原則に違反する | 最上位 Section の goal は学習者の実タスク上の成果物で記述する。入れ子 Section の goal で「なぜこの小手順を行うのか」を実タスク文脈で説明する |
| **ミニマリズム P3: エラー認識・回復の支援** (van der Meij & Carroll, 1995) | エラーは学習機会であり、予防・検出・診断・回復の全段階を支援する | Recovery コンポーネントは**回復手順のみ**を扱う。予防（操作前の注意喚起）と検出（エラー症状の記述）も別途必要 | Recovery に加えて、失敗しやすい操作の**直前**に予防的注意を置く。Recovery 内では「症状→原因→回復手順」の3段構成で書く |
| **ミニマリズム P4: 柔軟な利用の支援** (van der Meij & Carroll, 1995) | 学習者は文書を最初から順に読まない。拾い読み・飛ばし読み・逆引きを支援する | 本スキルの主対象は**順序付きチュートリアル**であるため、完全な非線形設計は求めない。ただし各 Step は可能な限り自己完結させる | 各 Step 冒頭の goal で「この Step で何ができるようになるか」を宣言し、途中参入者が必要な Step を特定できるようにする。Concept と Reference を折りたたみにして既知の読者がスキップできるようにする |
| **シグナリングの原理** (Mayer, 2009) | 重要箇所を視覚的手がかりで強調すると注意配分が改善し本質処理に集中できる | 合図は**学習目的に沿った要素**にのみ付ける。装飾目的の強調・感情表現の太字は一貫性原理違反 | Section の goal 宣言、画像の番号吹き出し、太字 UI 要素名、①②③ の順序番号で重要箇所を示す |
| **事前トレーニングの原理** (Mayer, 2009) | キー用語の名前と特徴を事前に提示すると主学習時の外在的処理が下がる | 予習は**これから出る概念のみ**に限定。遠い将来に出る概念や全体概論はミニマリズム違反 | Concept は first-use の直前に置き、「名前＋鍵となる特徴」を最小単位で提示する |
| **個人化の原理** (Mayer, 2009) | 会話的・二人称・能動的文体はフォーマル文体より学習効果が高い | 文体の**親しみやすさ**が本質。馴れ馴れしさ・絵文字濫用・感情過剰は一貫性原理違反になり得る | 学習者に直接語りかける二人称・能動形で書く（日本語：「〜しましょう」「確認してください」）。三人称で読者を描写しない（「受講者が〜する」「初学者向け」等を禁止） |
| **生成活動の原理** (Mayer, 2014) | 学習者に要約・予測・説明などの生成活動を求めると学習が深まる | 活動は**学習目的に関連**していること。単なる作業の追加は一貫性原理違反 | Verify で「何が起きるか」を観察判断させる。Checkpoint で behavior を自己確認させる。Exercise を周期的に織り込む |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [metyatech/skill-tutorial-authoring](https://github.com/metyatech/skill-tutorial-authoring) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
