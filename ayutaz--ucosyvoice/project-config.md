---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

**uCosyVoice** - CosyVoice3のモデルをONNX形式でUnity AI Interface（旧Sentis）で動作させるプロジェクト。対応言語は英語のみ。

### プロジェクト状態: ✅ 全フェーズ完了

| Phase | 内容 | 状態 |
|-------|------|------|
| Phase 1 | 環境構築・ONNX検証 | ✅ 完了 |
| Phase 2 | HiFT (Vocoder) C#実装 | ✅ 完了 |
| Phase 3 | Flow Matching C#実装 | ✅ 完了 |
| Phase 4 | LLM C#実装 | ✅ 完了 |
| Phase 5 | テキスト処理・トークナイザー | ✅ 完了 |
| Phase 6 | プロンプト音声処理 | ✅ 完了 |
| Phase 7 | 統合・最適化 | ✅ 完了 |

**テスト**: 全118テスト合格

### サンプルシーン
- `Assets/uCosyVoice/Samples/TTSSampleScene.unity` - すぐに使えるTTSデモ

### 参照リポジトリ
- **CosyVoice**: `C:\Users\yuta\Desktop\Private\CosyVoice`（元のPyTorch実装）

---

## 使用方法

### 基本的なテキスト→音声変換

```csharp
using uCosyVoice.Core;
using UnityEngine;

// 初期化
var manager = new CosyVoiceManager();
manager.Load(BackendType.CPU);

// テキスト→音声
float[] audio = manager.Synthesize("Hello, world!");
AudioClip clip = manager.CreateAudioClip(audio);

// 再生
audioSource.clip = clip;
audioSource.Play();

// 解放
manager.Dispose();
```

### サンプルシーンの使い方

1. `Assets/uCosyVoice/Samples/TTSSampleScene.unity` を開く
2. Play Modeを開始
3. "Load Models" ボタンをクリック（初回読み込みに時間がかかります）
4. テキストを入力して "Synthesize" をクリック
5. 音声が再生されます

### 音声クローニング（ゼロショット）

```csharp
// プロンプトモデルをロード
manager.LoadPromptModels();

// 参照音声（16kHz）と書き起こしテキストを使用（推奨）
float[] promptAudio = ...; // 16kHzの参照音声
string promptText = "Hello, my name is Sarah."; // 参照音声の書き起こし
float[] audio = manager.SynthesizeWithPrompt(
    "Nice to meet you!",    // 生成したいテキスト
    promptText,              // 参照音声の書き起こし
    promptAudio              // 16kHzの参照音声
);

// 簡易版（promptTextなし、品質は低下）
float[] audio2 = manager.SynthesizeWithPrompt("Hello!", promptAudio);

// 話者埋め込みを抽出して再利用
float[] embedding = manager.ExtractSpeakerEmbedding(promptAudio);
manager.SetDefaultSpeakerEmbedding(embedding);
float[] audio3 = manager.Synthesize("Another sentence.");
```

**重要**: ゼロショットTTSの品質を最大化するには、`promptText`（参照音声の書き起こし）を必ず提供してください。

---

## 制約事項

### 言語制約

| 項目 | 制約 |
|------|------|
| **対応言語** | **英語のみ** |
| テキスト正規化 | 英語の略語、数字、通貨のみ対応 |

### 音声入力制約（ゼロショットTTS用）

| 項目 | 値 | 備考 |
|------|-----|------|
| **参照音声サンプルレート** | 16kHz必須 | 他のレートは自動リサンプル |
| **SpeechTokenizer最大長** | 30秒 | `MAX_AUDIO_LENGTH_SEC = 30` |
| **CAMPPlus話者エンコーダ最大フレーム** | 200フレーム（約2秒） | 位置エンコーディング制限により自動切り詰め |

> **注意**: 参照音声が2秒を超える場合、話者埋め込み抽出時に最初の約2秒に自動切り詰められます。

### テキスト・トークン制約

| 項目 | デフォルト値 | 設定可能 |
|------|-------------|----------|
| 最大生成トークン | 500 | `MaxTokens` |
| 最小生成トークン | 10 | `MinTokens` |
| Top-kサンプリング | 25 | `SamplingK` |

実際の生成長は入力テキストトークン数に基づいて自動調整されます：
- `minLen = max(minLen, textTokens × 2)`
- `maxLen = min(maxLen, textTokens × 20)`

### モデルサイズとメモリ制約

| カテゴリ | サイズ |
|----------|--------|
| LLMモデル合計 | 約1.5GB |
| Flowモデル合計 | 約666MB |
| HiFTモデル合計 | 約342MB |
| プロンプトモデル合計 | 約997MB |
| **合計** | **約4GB** |

**推奨メモリ**: 8GB RAM以上

### バックエンド制約

| バックエンド | 推奨度 | 備考 |
|-------------|--------|------|
| **CPU** | ✅ 推奨 | 全モデルで安定動作 |
| **GPUCompute** | ⚠️ 条件付き | FP32モデルで精度問題の可能性 |

### パフォーマンス制約

| 処理 | 特性 |
|------|------|
| モデルロード | 初回約10-30秒 |
| **推論** | **ブロッキング処理（UIフリーズあり）** |
| LLM生成 | 自己回帰ループ（トークンごとに順次処理） |

> **注意**: 合成処理中はUIがフリーズします。非同期処理は`LoadAsync`のみ対応。

### 出力音声

| 項目 | 値 |
|------|-----|
| 出力サンプルレート | 24kHz |
| 音声クリッピング | ±0.99 |

### 未対応機能（Python版との差分）

- 多言語対応（中国語等）
- ストリーミング合成
- リアルタイム音声録音
- インストラクトモード
- 非同期推論（合成処理）

---

## アーキテクチャ

### 処理パイプライン（基本）

```
入力テキスト
    ↓
[TextNormalizer] テキスト正規化（数字展開、略語など）
    ↓
[Qwen2Tokenizer] BPEトークン化
    ↓
[LLMRunner] 自己回帰で音声トークン生成
├── Text Embedding
├── Speech Embedding (SOS, TASK_ID)
├── Backbone Initial → KVキャッシュ生成
└── Backbone Decode (自己回帰ループ)
    ↓
[FlowRunner] DiT + Euler Solver
├── Token Embedding
├── Pre-Lookahead
├── Speaker Projection
└── 10ステップODE積分 → メルスペクトログラム
    ↓
[HiFTInference] ボコーダー
├── F0 Predictor → F0推定
├── Source Generator → ソース信号
├── MiniSTFT (n_fft=16, hop=4)
├── Decoder → マグニチュード/位相
└── MiniISTFT → 波形
    ↓
[AudioClipBuilder] 24kHz音声 → AudioClip
```

### ゼロショットTTSパイプライン

```
プロンプト音声 [16kHz] + プロンプトテキスト + 生成テキスト
    ↓
[プロンプト処理]
├── [KaldiFbank] → [SpeakerEncoder CAMPPlus] → speaker_emb [1, 192]
├── [WhisperMelExtractor] → [SpeechTokenizer] → prompt_speech_tokens [1, T']
└── [FlowMelExtractor] (24kHz変換後) → prompt_mel [1, 80, T'']
    ↓
[LLMRunner] ゼロショットモード
├── prompt_text + tts_text を連結してトークン化
├── LLM入力: [SOS, combined_text_emb, TASK_ID, prompt_speech_emb]
└── 自己回帰で音声トークン生成 → generated_tokens
    ↓
[FlowRunner] プロンプト条件付き生成
├── all_tokens = prompt_tokens + generated_tokens
├── conds にprompt_melを埋め込み
├── 10ステップODE積分
└── 出力から prompt 部分を除去 → generated_mel のみ
    ↓
[HiFTInference] → [AudioClipBuilder] → 24kHz音声
```

---

## ファイル構成

```
Assets/
├── Models/                              # ONNXモデル（14ファイル）
│   ├── text_embedding_fp32.onnx
│   ├── llm_backbone_initial_fp16.onnx
│   ├── llm_backbone_decode_fp16.onnx
│   ├── llm_decoder_fp16.onnx
│   ├── llm_speech_embedding_fp16.onnx
│   ├── flow_token_embedding_fp16.onnx
│   ├── flow_pre_lookahead_fp16.onnx
│   ├── flow_speaker_projection_fp16.onnx
│   ├── flow.decoder.estimator.fp16.onnx
│   ├── hift_f0_predictor_fp32.onnx
│   ├── hift_source_generator_fp32.onnx
│   ├── hift_decoder_fp32.onnx
│   ├── campplus.onnx                    # 話者エンコーダ

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ayutaz/uCosyVoice](https://github.com/ayutaz/uCosyVoice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
