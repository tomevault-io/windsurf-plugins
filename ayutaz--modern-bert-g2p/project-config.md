---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト目的

ModernBERT (encoder-only pure-NN) の日本語 G2P (Grapheme-to-Phoneme) 応用可能性を検証する **研究プロジェクト**。推論パスに rule / dictionary lookup を含めない **pure-NN, no dict inference** を厳守する。Primary target は公開されている pure-NN Japanese G2P モデル群を JSUT / JVS / ROHAN 標準ベンチで系統的に上回ること。

### 3ティアの competitive landscape

1. **Primary (must-beat, pure-NN のみ)**: 越えるべき先行 pure-NN 4 モデル
   - **CharsiuG2P** (ByT5-small, ~300M, 多言語) — 自著 held-out で日本語 PER 10.51% (IPA-vs-dict word-list; monolingual-JA-only ByT5 は 66.89% に崩壊)
   - **PnG BERT (Yasuda & Toda 2022)** — 事前学習 validation の whole-word G2P accuracy 45.5% のみ公表 (JSUT 別途 PER は未公表)。TTS accent MOS 2.51 vs rule-labeled Tacotron 3.04 で敗北
   - **Kakegawa TJ-G2P (Kurihara & Sano 2024 の再実装)** — JSUT400 (Kurihara が Basic5000 から 400 抽出した ad-hoc split) で PPL CER 11.85% (OpenJTalk 10.82% に敗北)。Kakegawa 2021 原論文は JSUT を触っておらず新聞 5,142 文で Word-Accuracy 評価
   - **CC-G2PnP (Shirahata & Yamamoto 2026, SB Intuitions)** — 内部 6D-Eval (2,722 文) で PnP CER 1.79-1.80%、Phoneme CER 0.48-0.52%。Dict-DNN-NS hybrid が 1.71%/0.40% で勝つ (「hybrid 越え」は refute 済み)
2. **Reference (compare but not target)**: haqumei 1.17% / OpenJTalk 1.03% — hybrid 天井なので pure-NN 130M では絶対値では劣る前提。gap を可視化するが acceptance criteria には入れない
3. **Aspirational (world-first if achieved)**: Frontier LLM (Claude Opus 4.6 0.52%, Gemini 3.1 Pro 0.62% on JVS-3000) を 130M pure-NN で近似できるか。LLM の parse mode 依存度が未定量のため文字通りの越えは非現実的想定

### Pivot の理由 (v2.0, 2026-07-04)

「ルールベースの G2P 処理が入るのであればこのプロジェクトをする必要がない」 — hybrid 設計は結果の 80-95% が辞書由来のため、130M ModernBERT を書く動機自体が消える。pure-NN 世界内の空白領域を JSUT / JVS / ROHAN 標準ベンチで埋める初のデータポイントとして再定義する。

## リポジトリの現状

- **コード未実装** — ソースコード、テスト、CI等は未着手
- **調査完了** — 9本の技術ドキュメントが `docs/research/` に存在
- **既存ファイル**:
  - `docs/requirements.md` — **要求定義書 (要件ID FR/NFR/CR/AC付き、v2.0 で pure-NN pivot 反映済み)**
  - `docs/research/01_overview.md` — サマリーと開発戦略
  - `docs/research/02_existing_systems.md` — 既存G2Pシステムのサーベイ
  - `docs/research/03_datasets_and_benchmarks.md` — 評価データ・ベンチマーク・ライセンス
  - `docs/research/04_papers_and_references.md` — 論文と一次情報
  - `docs/research/05_technical_design.md` — ModernBERT ベースのモデル設計
  - `docs/research/06_implementation_roadmap.md` — 実装ロードマップ (v2.0 で P4 hybrid 削除、pretrain-plus-fine-tune に置換)
  - `docs/research/07_nn_only_benchmarks.md` — 純粋NN日本語G2Pのベンチマーク徹底調査 (v2.0 で「越えるべき baseline」に framing shift)
  - `docs/research/08_market_landscape.md` — 市場に存在する日本語対応NN-G2Pモデル 網羅的インベントリ
  - `docs/research/09_pure_nn_g2p_benchmarks.md` — 越えるべき pure-NN 先行 4 モデルの徹底整理

## 設計の核心思想 (実装時の判断基準)

以下は pure-NN pivot 後の設計原則。実装時に迷ったらこの原則に戻る:

1. **Pure-NN G2P として学習・推論する**。dict lookup / MeCab 出力を **推論パスに含めない**。ただし char-level BERT (P-C) の pre-tokenize は「内部トークナイザー」として許容し、pyopenjtalk / MeCab を推論に呼ぶことはしない。
2. **NHK Kurihara Interspeech 2024 の TJ-G2P + BAS の architecture は継承**。T5 seq2seq (encoder) と char-BERT (BAS) の 2 段を、hybrid の post-processor としてではなく **multi-task の 1 head として encoder に統合** する。
3. **Hida ICASSP 2022 のマルチタスク (G2P + 多音字 + APBP + ANPP)** を multi-head で実装する。主観 MOS 3.67 (対 オラクル 3.69) の near-oracle 品質を目指す根拠。BAS を 5 つ目の head に追加する。
4. **SentencePiece トークナイザーの警告に注意**: SB Intuitions 自らが「token classification タスクで性能が悪い」と modernbert-ja HFカードで明記。naive per-token classification は避け、seq2seq か char-level BERT のどちらかを Phase 2 で head-to-head 比較して選ぶ (2 pilot P-A/P-C 並列)。**P-B (MeCab pretokenize) は v2.0 pivot で drop — MeCab 依存が pure-NN 原則に反するため**。
5. **Pure-NN の実証的失敗パターン (docs/research/07 の 5 事例、docs/research/09 の追加分析) は既知の baseline**。同じ失敗を避けるため、以下 3 点を学習前に固定する:
   - (a) **データ量**: PnG BERT (Aozora 4.9M) / Kakegawa (news 5M) より 5-10 倍以上のコーパス (5M+ sentence-level pairs、char-level では 50M+ token) を確保
   - (b) **マルチタスク supervision (Hida 2022 型)** を pretrain と fine-tune の両方で活用
   - (c) **評価は JSUT / JVS / ROHAN 3 本柱に統一** (先行 pure-NN は指標が異なり比較不能だった)

### 補足: pyopenjtalk-plus (surface, yomi, accent) の扱い

- **教師信号としての利用は許容**: MLM 継続 pretrain / 事前学習 pseudo-label としては使う (PnG BERT が Aozora + morph 解析 pseudo-label で pretrain したのと同じ慣行)
- **推論時の利用は禁止**: 推論パスに pyopenjtalk / MeCab / 辞書 lookup を呼ばない
- **既知の限界**: 辞書由来の教師信号は辞書エラーを NN が継承する rule-leakage である。Frontier LLM が 0.52% を出せるのは web-scale の非辞書コーパスがあるからで、我々の 130M では辞書 pretrain が実質的な上限を作りうる。この点は `docs/research/09` §7 で明示的に self-audit する

## 主要な数値目標

Pure-NN research の tier 設計。**Reference values (haqumei 1.17%, OpenJTalk 1.03%, Claude Opus 4.6 0.52%) は表の脚注で参照値として扱い、acceptance criteria には含めない**。

| 指標 | Pure-NN baseline (先行研究) | Conservative | Stretch | Aspirational |
|---|---|---|---|---|
| JSUT Basic5000 PER | CharsiuG2P 10.51% (自著 held-out, IPA) | < 5.0% | < 3.0% | < 0.6% (Frontier LLM 並み) |
| JVS-3000 kana CER | pure-NN 公開値なし (Phase 2 で自己測定; Koriyama Interspeech 2026 arxiv 2606.22009 が benchmark 定義) | < 5.0% | < 2.0% | < 0.62% (Gemini 3.1 Pro 越え) |
| ROHAN KER | pure-NN 公開値なし (Phase 2 で自己測定) | < 5.0% | < 2.0% | < 1.0% |
| JSUT モーラアクセント精度 | PnG BERT の whole-seg 45.5% からは換算不能 | > 90% | > 96.66% (Hida hybrid 越え) | > 98% |
| Multi-head compatibility (Phase 3) | 該当なし (先行 pure-NN は single-task) | G2P / polyphone / APBP / ANPP / BAS の 5 head を joint training し、G2P 単独 fine-tune 比 PER 相対改善 > 5% | > 15% | > 30% |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ayutaz/modern-bert-g2p](https://github.com/ayutaz/modern-bert-g2p) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
