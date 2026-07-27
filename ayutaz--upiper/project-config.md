---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

uPiperは[piper-plus](https://github.com/ayutaz/piper-plus)ベースの高品質ニューラルTTS（Text-to-Speech）Unityプラグイン。VITS（Variational Inference with adversarial learning for end-to-end Text-to-Speech）モデルを使用。G2Pは7言語対応、現行モデルは6言語（ja/en/zh/es/fr/pt）に対応。

| 言語 | G2Pバックエンド |
|------|----------------|
| 日本語 | DotNetG2P.MeCab (dot-net-g2p / MeCab辞書) |
| 英語 | DotNetG2P.English (EnglishG2PEngine, CMU dict + LTS) |
| スペイン語 | DotNetG2P.Spanish (SpanishG2PEngine) |
| フランス語 | DotNetG2P.French (FrenchG2PEngine) |
| ポルトガル語 | DotNetG2P.Portuguese (PortugueseG2PEngine) |
| 中国語 | DotNetG2P.Chinese (ChineseG2PEngine, 44K文字辞書) |
| 韓国語 | DotNetG2P.Korean (KoreanG2PEngine) |

### 対応モデル

| モデル名 | 言語 | Prosody対応 | 説明 |
|---------|------|------------|------|
| multilingual-test-medium | 多言語(6言語) | Yes | 多言語対応モデル（ja/en/zh/es/fr/pt）、fp16、38MB、`phoneme_type: "multilingual"` |

## ビルド・テストコマンド

### Unity テスト実行
```bash
# GitHub Actions経由（EditMode + PlayModeテスト）
# .github/workflows/unity-tests.yml 参照

# Unity Editor内
# Window > General > Test Runner > Run All
```

### コードフォーマットチェック
```bash
dotnet format --verify-no-changes
```

## アーキテクチャ

### データフロー
```
テキスト入力
    ↓
カスタム辞書による前処理 (CustomDictionary)
    • 技術用語・固有名詞の読み変換
    • 例: "Docker" → "ドッカー", "GitHub" → "ギットハブ"
    ↓
MultilingualPhonemizer (言語ルーティング, DotNetG2Pエンジン直接呼び出し)
    ├─ ja: DotNetG2PPhonemizer (dot-net-g2p, MeCab辞書)
    ├─ en: EnglishG2PEngine (DotNetG2P.English, CMU dict + LTS)
    ├─ es: SpanishG2PEngine (DotNetG2P.Spanish)
    ├─ fr: FrenchG2PEngine (DotNetG2P.French)
    ├─ pt: PortugueseG2PEngine (DotNetG2P.Portuguese)
    ├─ zh: ChineseG2PEngine (DotNetG2P.Chinese, 44K文字辞書)
    └─ ko: KoreanG2PEngine (DotNetG2P.Korean)
    ↓
PuaTokenMapper (PUA↔IPA双方向マッピング, 87固定エントリ)
    • 全7言語の音素をPUA文字にマッピング
    ↓
音素エンコーディング (Unicode PUAマッピング)
    • 複数文字音素（ky, ch, ts, sh等）→ Private Use Area文字
    ↓
┌──────────────────────────────────────────────────────┐
│ Prosody対応モデルの場合                               │
│   Prosody情報取得 (言語別)                           │
│     • ja: A1=モーラ位置, A2=アクセント核, A3=句位置  │
│     • en: A1=0, A2=0, A3=0                           │
│     • zh: A1=tone(1-5), A2=音節位置, A3=単語長       │
│     • ko: A1=0, A2=0, A3=音節数                      │
│     • es/fr/pt: A1=0, A2=stress(0/2), A3=語内音素数  │
└──────────────────────────────────────────────────────┘
    ↓
┌──────────────────────────────────────────────────────┐
│ 多言語モデルの場合                                    │
│   • Auto-promotion: phoneme_type:"multilingual"の     │
│     モデルは自動的にMultilingualPhonemizerを生成       │
│   • PhonemeEncoder後にIntersperse PAD挿入:            │
│     [^, _, p1, _, p2, _, ..., $]                      │
│     （BOS後にもPADが入る）                             │
└──────────────────────────────────────────────────────┘
    ↓
VITS推論 (ONNX via Unity.InferenceEngine)
    • GPU: GPUPixel (推奨), GPUCompute (非推奨)
    • CPU: macOSデフォルト
    • Prosody対応: GenerateAudioWithProsodyAsync
    ↓
AudioClip出力 (22050Hz, float32)
```

### 主要コンポーネント

| コンポーネント | 場所 | 役割 |
|--------------|------|------|
| `IPiperTTS` / `PiperTTS` | `Runtime/Core/` | メインインターフェース |
| `IPhonemizerBackend` | `Runtime/Core/Phonemizers/Backend/` | 音素化バックエンド抽象（テストスタブのみ使用） |
| `MultilingualPhonemizer` | `Runtime/Core/Phonemizers/Multilingual/` | 多言語テキスト分割・DotNetG2Pエンジン直接呼び出し |
| `UnicodeLanguageDetector` | `Runtime/Core/Phonemizers/Multilingual/` | Unicode文字範囲ベース言語検出 |
| `DotNetG2PPhonemizer` | `Runtime/Core/Phonemizers/Implementations/` | 日本語G2P（dot-net-g2p, Prosody対応） |
| `CustomDictionary` | `Runtime/Core/Phonemizers/` | カスタム辞書（技術用語・固有名詞の読み変換） |
| `PiperConfig` | `Runtime/Core/` | 設定管理（GPU, キャッシュ, バックエンド選択） |
| `AudioChunkBuilder` | `Runtime/Core/AudioGeneration/` | 音声波形→AudioClip変換 |
| `InferenceAudioGenerator` | `Runtime/Core/AudioGeneration/` | ONNX直接推論（Prosody対応） |
| `EnglishG2PEngine` | DotNetG2P.English パッケージ | 英語G2P（CMU dict + LTS + 同音異義語解決） |
| `SpanishG2PEngine` | DotNetG2P.Spanish パッケージ | スペイン語G2P |
| `FrenchG2PEngine` | DotNetG2P.French パッケージ | フランス語G2P |
| `PortugueseG2PEngine` | DotNetG2P.Portuguese パッケージ | ポルトガル語G2P |
| `ChineseG2PEngine` | DotNetG2P.Chinese パッケージ | 中国語G2P（44K文字辞書） |
| `KoreanG2PEngine` | DotNetG2P.Korean パッケージ | 韓国語G2P（Hangul分解 + 音韻規則） |
| `PuaTokenMapper` | `Runtime/Core/Phonemizers/Multilingual/` | PUA↔IPA双方向マッピング |
| `LanguageConstants` | `Runtime/Core/Phonemizers/Multilingual/` | 言語ID/コード定数 |
| `InferenceEngineDemo` | `Runtime/Demo/` | テスト用デモUI（6言語ドロップダウン） |

### ディレクトリ構造
```
Assets/uPiper/
├── Runtime/
│   ├── Core/               # ランタイムコア
│   │   ├── AudioGeneration/    # AudioClip生成、ONNX推論
│   │   ├── Phonemizers/        # 音素化システム
│   │   │   ├── Backend/        # バックエンドインターフェース・共有型（IPhonemizerBackend, PhonemeOptions）
│   │   │   ├── Implementations/# Prosody対応実装
│   │   │   ├── Multilingual/   # 多言語共通(PuaTokenMapper, LanguageConstants)
│   │   │   ├── Native/         # P/Invoke定義
│   │   │   └── Threading/      # マルチスレッド処理
│   │   ├── IL2CPP/             # IL2CPP互換レイヤー
│   │   └── Platform/           # プラットフォーム固有コード
│   │       ├── WebGLStreamingAssetsLoader.cs  # WebGL非同期ファイルローダー
│   │       ├── IndexedDBCache.cs              # IndexedDBキャッシュC#ラッパー

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ayutaz/uPiper](https://github.com/ayutaz/uPiper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
