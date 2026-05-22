---
trigger: always_on
description: AviUtlの代替を目指したRust製オープンソース動画編集ソフトウェア。
---

# CLAUDE.md - RuViE (Rust Video Editor)

## プロジェクト概要

AviUtlの代替を目指したRust製オープンソース動画編集ソフトウェア。
Skia (GPU/OpenGL) によるリアルタイムプレビュー、FFmpegベースのメディア処理、eguiによるデスクトップGUIを持つ。

## ワークスペース構成

```
Cargo workspace (3 members)
├── library/    … コアライブラリ (edition 2024) - レンダリング、音声、プラグイン、モデル
├── app/        … GUIアプリケーション (edition 2021) - egui/eframe ベースのエディタUI
└── plugins/random_property/ … サンプルプラグイン (cdylib)
```

## ビルド・実行コマンド

```bash
# GUIアプリ起動
cargo run -p app

# リリースビルド
cargo build -p app --release

# CLIでレンダリング (ヘッドレス)
cargo run -p library -- test_data/project.json

# プラグインビルド (DLL/so)
cargo build -p random_property_plugin

# テスト (ignored テストを含む)
cargo test -p library -- --ignored

# 通常テスト
cargo test -p library
```

## アーキテクチャ

### レイヤー構造

```
app (GUI) → library (core)
  ├── editor/   … サービス層 (EditorService, RenderService, ExportService, AudioService)
  │   └── handlers/ … 操作ハンドラ (clip, track, asset, keyframe, composition, property)
  ├── model/    … データモデル (Project, Composition, Node, Clip, Property, Effect)
  ├── core/     … 内部処理
  │   ├── rendering/ … Skiaベースレンダリング (RenderServer: バックグラウンドスレッド)
  │   ├── audio/     … cpal + symphonia による音声再生・ミキシング
  │   ├── cache/     … LRUキャッシュ
  │   ├── ensemble/  … テキストアニメーション (エフェクター・デコレーター)
  │   └── framing/   … フレーム・領域管理
  └── plugin/   … プラグインシステム
      ├── effects/          … 組み込みエフェクト (blur, dilate, erode, drop_shadow, magnifier, tile, pixel_sorter, sksl)
      ├── loaders/          … メディアローダー (画像, FFmpeg動画)
      ├── exporters/        … エクスポーター (PNG連番, FFmpeg動画)
      ├── properties/       … プロパティ評価 (constant, keyframe, expression)
      ├── entity_converter/ … エンティティ→描画変換 (image, video, text, shape, sksl)
      ├── decorators.rs     … テキストデコレーター
      ├── effectors.rs      … テキストエフェクター
      └── styles.rs         … スタイルプラグイン
```

### app GUI構造

```
app/src/
├── main.rs        … エントリーポイント (eframe, 1920x1080)
├── app.rs         … RuViEApp (メインアプリ状態)
├── command.rs     … コマンドレジストリ (NewProject, Save, Undo, Redo, etc.)
├── shortcut.rs    … キーボードショートカット管理
├── config.rs      … 設定ファイル管理
├── action/        … アクション/コマンドハンドラ
├── state/         … EditorContext (タイムライン/ビュー/選択/インタラクション状態)
├── model/         … UI固有モデル (ノードグラフ, ベクター)
├── ui/
│   ├── panels/
│   │   ├── preview/    … プレビューパネル (ギズモ, グリッド, ベクターエディタ)
│   │   ├── timeline/   … タイムラインパネル (ルーラー, クリップエリア, コントロール)
│   │   ├── inspector/  … インスペクターパネル (プロパティ, エフェクト, スタイル)
│   │   ├── assets.rs   … アセットパネル
│   │   ├── node_editor.rs
│   │   └── graph_editor/ … キーフレームグラフ
│   ├── dialogs/    … モーダルダイアログ (export, composition, keyframe, settings)
│   ├── widgets/    … 再利用ウィジェット (modal, reorderable_list, searchable_context_menu)
│   ├── viewport.rs … ビューポートレンダリング
│   ├── menu.rs     … メニューバー
│   └── theme.rs    … Catppuccin テーマ
└── utils/
```

### 主要データモデル

```
Project
├── composition_ids: Vec<Uuid>               … コンポジション順序
├── assets: Vec<Asset>                       … メディアファイル
├── nodes: HashMap<Uuid, Node>               … 統合ノードレジストリ (全ノード)
├── connections: Vec<Connection>             … ノード間接続
└── export: ExportConfig

Node (enum, #[serde(tag = "node_type")])
├── Track(TrackData)        … トラック (child_ids で子ノード参照)
├── Layer(LayerData)        … レイヤーコンテナ (旧 is_layer トラック)
├── Composition(Composition)… コンポジション (root_track_id, fps, size, duration)
├── Source(SourceData)      … メディアソース (#[serde(alias = "Clip")])
│   ├── kind: SourceKind (Video, Audio, Text, Shape, SkSL, Image, Composition)
│   ├── properties: PropertyMap
│   └── in_frame, out_frame, source_begin_frame, fps
└── Graph(GraphNode)        … グラフノード (エフェクト, トランスフォーム, スタイル等)
```

### 主要デザインパターン

- **Arc<RwLock<T>>**: Project や AudioEngine などの共有状態
- **サービス層**: EditorService がファサードとして GUI とコアを分離
- **コマンドパターン**: CommandId → handle_command() によるアクション処理
- **HistoryManager**: Project 全体のクローンによる undo/redo
- **バックグラウンドレンダリング**: RenderServer がチャネル経由でリクエスト受信・レンダリング
- **プラグインシステム**: libloading による動的ライブラリ読み込み + trait ベース

## 主要依存ライブラリ

| ライブラリ | 用途 |
|-----------|------|
| skia-safe | 2Dグラフィックスレンダリング (GPU/GL) |
| ffmpeg-next | 動画・音声コーデック処理 |
| egui / eframe / egui_dock | GUI フレームワーク |
| cpal + symphonia + rubato | 音声再生・デコード・リサンプリング |
| tokio | 非同期ランタイム |
| rayon | 並列処理 |
| pyo3 | Python式評価 (エクスプレッションプラグイン) |
| glutin + winit | OpenGLコンテキスト・ウィンドウ管理 |
| serde + serde_json + bincode | シリアライズ (プロジェクトJSON) |
| shaderc | GLSLシェーダーコンパイル |

## 外部依存

- `external/OpenColorIO/` … 色空間管理ライブラリ (DLL)
- `external/shim/` … OpenColorIO C++ラッパー (shim.cpp)
- `OpenColorIO.dll`, `shim.dll` … プロジェクトルートに配置
- FFmpeg バイナリがシステム PATH 上に必要

## テスト

### テスト実行コマンド

```bash
# library ユニットテスト
cargo test -p library

# app UI テスト (egui_kittest)
cargo test -p app

# 全テスト
cargo test --workspace

# ignored テストを含む (E2E、リグレッション)
cargo test -p library -- --ignored

# スナップショット更新 (意図的な UI 変更後)
UPDATE_SNAPSHOTS=true cargo test -p app
```

### テスト構成

| 種類 | 場所 | フレームワーク | 実行方法 |
|------|------|---------------|----------|
| ユニットテスト | `library/src/` 内 `#[cfg(test)]` | 標準 | `cargo test -p library` |
| 統合テスト | `library/tests/` | 標準 | `cargo test -p library` |
| GUI テスト | `app/src/` 内 `#[cfg(test)]` | egui_kittest | `cargo test -p app` |
| E2E/リグレッション | `library/tests/` | 標準 (`#[ignore]`) | `cargo test -p library -- --ignored` |

- テストデータ: `test_data/` (画像、動画、音声、プロジェクトJSON)
- レンダリング出力: `./rendered/` ディレクトリ

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Liesegang/video-editor](https://github.com/Liesegang/video-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
