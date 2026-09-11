---
trigger: always_on
description: このファイルは、@webxr-jp/avatar-optimizer (3D モデル最適化ユーティリティライブラリ) を扱う際に Claude Code へのガイダンスを提供します。
---

# CLAUDE.md

このファイルは、@webxr-jp/avatar-optimizer (3D モデル最適化ユーティリティライブラリ) を扱う際に Claude Code へのガイダンスを提供します。

## 会話について

日本語で会話すること

## プロジェクト概要

**@webxr-jp/avatar-optimizer** は WebXR アプリケーション向けの 3D モデル最適化ユーティリティライブラリです。glTF-Transform ベースの軽量ライブラリで、React 依存がなくブラウザ環境で動作します。

## プロジェクト構成

### スタック

- **TypeScript** (5.0+): 型安全なユーティリティ開発
- **pnpm** (workspace): monorepo パッケージ管理
- **tsup** (8.0+): ビルドツール (ESM/CJS 出力)
- **Vitest** (2.0+): ライブラリ/ビューア双方のユニットテスト
- **Three.js / @pixiv/three-vrm**: debug-viewer パッケージでの VRM 描画

### ディレクトリ構成（pnpm monorepo）

```
packages/
├── avatar-optimizer/              # メインライブラリ (VRM最適化 + テクスチャアトラス)
│   ├── src/
│   │   ├── process/              # 最適化プロセス
│   │   │   ├── gen-atlas.ts       # テクスチャアトラス生成
│   │   │   ├── packing.ts         # パッキングアルゴリズム
│   │   │   └── set-uv.ts          # UV リマッピング
│   │   ├── util/                 # ユーティリティ
│   │   │   ├── material/          # マテリアル処理
│   │   │   │   ├── index.ts
│   │   │   │   ├── combine.ts
│   │   │   │   └── types.ts
│   │   │   ├── mesh/              # メッシュ処理
│   │   │   │   ├── merge-mesh.ts
│   │   │   │   ├── uv.ts
│   │   │   │   └── deleter.ts
│   │   │   └── texture/           # テクスチャ処理
│   │   │       ├── index.ts
│   │   │       ├── composite.ts
│   │   │       ├── packing.ts
│   │   │       └── types.ts
│   │   ├── avatar-optimizer.ts    # メイン API 実装
│   │   ├── index.ts               # ライブラリエクスポート管理
│   │   └── types.ts               # 型定義集約
│   ├── tests/                     # Vitest 自動テスト
│   │   ├── *.test.ts              # 最適化/アトラス/アダプタ検証
│   ├── dist/                      # ビルド出力 (ESM/型定義)
│   ├── package.json
│   ├── tsconfig.json
│   ├── vitest.config.ts
│   └── tsup.config.ts
│
├── mtoon-atlas/             # MToon Atlas マテリアル
│   ├── src/
│   │   ├── shaders/
│   │   │   ├── mtoon.frag
│   │   │   └── mtoon.vert
│   │   ├── MToonAtlasMaterial.ts
│   │   ├── declarations.d.ts
│   │   ├── index.ts
│   │   └── types.ts
│   ├── tests/                    # Vitest テスト
│   ├── dist/                     # ビルド出力
│   ├── package.json
│   ├── tsconfig.json
│   ├── vitest.config.ts
│   ├── tsup.config.ts
│   └── README.md
│
└── debug-viewer/                 # VRM ビューア (React + Three.js)
    ├── src/
    │   ├── components/            # React コンポーネント
    │   │   ├── VRMCanvas.tsx
    │   │   ├── VRMScene.tsx
    │   │   ├── VRMViewer.tsx
    │   │   ├── Viewport3D.tsx
    │   │   ├── TextureViewer.tsx
    │   │   ├── TexturePreviewScene.tsx
    │   │   ├── SceneInspector.tsx
    │   │   ├── UVPreviewDialog.tsx
    │   │   └── index.ts
    │   ├── hooks/                 # React カスタムフック
    │   │   ├── useVRMLoader.ts     # VRM 読み込み処理
    │   │   ├── useVRMScene.ts      # VRM シーン管理
    │   │   ├── useTextureReplacement.ts
    │   │   └── index.ts
    │   ├── assets/                # 静的アセット
    │   ├── App.tsx                # アプリケーションエントリー
    │   ├── main.tsx               # React マウントポイント
    │   └── index.css / App.css    # スタイル定義
    ├── dist/                      # ビルド出力
    ├── package.json
    ├── tsconfig.json
    ├── vitest.config.ts
    ├── vite.config.ts
    └── index.html

pnpm-workspace.yaml               # workspace 設定
package.json                       # ルート package.json (scripts 集約)
```

### 主要な API

#### ライブラリ API

- `optimizeVRM(file, options)`: テクスチャ圧縮・メッシュ削減による最適化
- `calculateVRMStatistics(file)`: VRM 統計計算 (ポリゴン数、テクスチャ数など)

詳細は `README.md` を参照してください。

## 開発コマンド

このプロジェクトは **pnpm monorepo** として構成されており、`packages/` ディレクトリ下に複数のパッケージが管理されています。

### ルートディレクトリでの操作

```bash
# 依存関係インストール（全ワークスペース）
pnpm install

# ビルド（全パッケージ）
pnpm build

# ウォッチモード（全パッケージ、開発時）
pnpm dev

# テスト実行（全パッケージ）
pnpm test

# Lint チェック
pnpm lint

# コード フォーマット
pnpm format
```

### 特定のパッケージ操作

```bash
# avatar-optimizer のビルド
pnpm -F avatar-optimizer run build

# avatar-optimizer の開発モード
pnpm -F avatar-optimizer run dev

# avatar-optimizer のテスト
pnpm -F avatar-optimizer run test

# mtoon-atlas (MToon インスタンシング)
pnpm -F mtoon-atlas run build
pnpm -F mtoon-atlas run dev
pnpm -F mtoon-atlas run test

# debug-viewer (VRM 確認用)
pnpm -F debug-viewer run build
pnpm -F debug-viewer run dev
pnpm -F debug-viewer run test
```

テクスチャアトラス機能は `packages/avatar-optimizer/src/process/` および `packages/avatar-optimizer/src/util/texture/` に分散して実装されています。`pnpm -F avatar-optimizer run test` がアトラス関連テストも実行します。

## 開発ルール

2. **モジュール形式**: 名前付きエクスポートを使用 (ESM 形式をサポート)
3. **テスト**: `packages/avatar-optimizer/tests/` や `packages/debug-viewer/tests/` で純粋関数のテストを記述

## AI 支援開発のためのコーディング規約

これらの規約は AI コード生成と自己修正向けに最適化されています。AI の誤りを防ぐため複雑性を制約しながらコード品質を維持します。

### 自己説明的なコード

コード再訪時に一貫性を維持するため、ファイルヘッダーに仕様コメントを含めます：

```typescript
/**
 * VRM モデルからテクスチャ統計を抽出して分析します。
 * baseColor テクスチャをスキャンし、
 * 総テクスチャメモリ使用量と圧縮可能性を評価します。
 *
 * @param document - glTF-Transform ドキュメント
 * @returns テクスチャ統計オブジェクト
 */
export function analyzeTextureStatistics(document: Document): TextureStats {
  // 実装
}
```

### 型集約 (真実の唯一の源)

**ドメインモデルを統合** して集約型ファイルでファイル読み込み削減：

```typescript
// ❌ 悪い例: 型がファイル全体に散在
// src/optimize.ts
export interface OptimizationOptions { ... }

// src/statistics.ts
export interface StatisticsResult { ... }

// ✅ 良い例: src/types.ts に型を集約
// src/types.ts
export interface OptimizationOptions { ... }
export interface StatisticsResult { ... }

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WebXR-JP/avatar-optimizer](https://github.com/WebXR-JP/avatar-optimizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
