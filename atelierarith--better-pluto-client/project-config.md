---
trigger: always_on
description: Pluto.jl リアクティブノートブックの機能を、ブラウザではなく VS Code の UI で表示・操作するための拡張機能。
---

# BetterPlutoClient - VS Code Extension

## プロジェクト概要

Pluto.jl リアクティブノートブックの機能を、ブラウザではなく VS Code の UI で表示・操作するための拡張機能。

**リポジトリ**: https://github.com/AtelierArith/better-pluto-client

## 目標

- Pluto.jl のバックエンド（Julia サーバー）に接続し、セル実行・リアクティビティを実現
- ブラウザを開かずに VS Code 内で完結したノートブック体験を提供
- Pluto.jl の既存機能（パッケージ管理、リアクティブ実行、エラー表示など）を活用

## 現在のアーキテクチャ

```
┌─────────────────────────────────────────────────────────┐
│ VS Code Extension                                        │
│  ┌─────────────────────┐  ┌───────────────────────────┐ │
│  │ PlutoNotebook       │  │ PlutoServer               │ │
│  │ Controller          │─▶│ (WebSocket)               │ │
│  │                     │  │                           │ │
│  │ - セル実行管理      │  │ - Pluto.jl プロセス管理   │ │
│  │ - 出力レンダリング  │  │ - WebSocket通信           │ │
│  │ - 状態管理          │  │ - MessagePack encode/decode│ │
│  └─────────────────────┘  └───────────────────────────┘ │
│            │                          │                  │
│            ▼                          │                  │
│  ┌─────────────────────┐              │                  │
│  │ pluto-renderer      │              │                  │
│  │ (Custom Renderer)   │              │                  │
│  │                     │              │                  │
│  │ - HTML出力表示      │              │                  │
│  │ - MathJax数式対応   │              │                  │
│  │ - インタラクティブ  │              │                  │
│  │   ウィジェット対応  │◀─────────────┘                  │
│  │ - Plotly/JS実行     │    (bond updates)              │
│  └─────────────────────┘                                │
└─────────────────────────────────────────────────────────┘
                             │
                             ▼ WebSocket (MessagePack)
┌─────────────────────────────────────────────────────────┐
│ Pluto.jl Server (Julia)                                  │
│  - セル実行                                              │
│  - リアクティブ依存関係管理                               │
│  - パッケージ管理                                        │
│  - エラーハンドリング                                    │
└─────────────────────────────────────────────────────────┘
```

## 実装タスク

### Phase 1: Pluto.jl サーバー接続 ✅
- [x] Pluto.jl サーバーの起動/管理 (`PlutoServer.ts`)
- [x] WebSocket クライアント実装 (MessagePack + ws)
- [x] メッセージプロトコル理解・実装
- [x] VS Code Notebook API との統合
- [ ] エラーハンドリング改善
- [ ] 接続状態の UI 表示

### Phase 2: UI 連携 ✅
- [x] セル実行結果の表示（Pluto形式）
- [x] エラー表示（Pluto.jl フォーマット）
- [x] リアクティブ更新の反映
- [x] 実行中状態の表示
- [x] インタラクティブウィジェット対応（PlutoUI: Slider, Select, Checkbox, Clock 等）
- [x] Pluto.jl 同一のツリー出力表示（折りたたみ可能）
- [x] 画像出力（PNG, SVG）
- [x] MathJax 3 による数式レンダリング
- [x] セル折りたたみ（code_folded）対応
- [x] Plotly/JS ベースのビジュアライゼーション（スクリプト実行対応）
- [x] 専用出力チャンネル（"BetterPlutoClient"）
- [x] 内部セル（TOML パッケージメタデータ）のラウンドトリップ保存
- [x] スマートセル同期（変更されたセルのみ更新送信）
- [x] 起動時セル実行の改善（VS Code 実行オブジェクト生成）
- [x] `last_run_timestamp` による実行完了検出

### Phase 3: 完全な機能
- [ ] パッケージ管理 UI
- [ ] 変数エクスプローラー
- [ ] ドキュメント表示
- [ ] セル依存関係グラフ表示
- [ ] プロットライブラリ完全対応（Plots.jl 等）

## 拡張機能コマンド・キーバインド

| コマンド | タイトル | キーバインド | アイコン |
|---------|--------|------------|---------|
| `pluto-notebook.openAsPlutoNotebook` | Open as Pluto Notebook | - | - |
| `pluto-notebook.startKernel` | Start Pluto Kernel | - | `$(play)` |
| `pluto-notebook.stopKernel` | Stop Pluto Kernel | - | `$(stop)` |
| `pluto-notebook.restartKernel` | Restart Pluto Kernel | - | `$(refresh)` |
| `pluto-notebook.wrapInBeginEnd` | Wrap Cell in begin...end | `Cmd+Shift+B` | `$(code)` |
| `pluto-notebook.saveAndExecute` | Save and Execute Modified Cells | `Cmd+S` | - |
| `pluto-notebook.toggleCellFolded` | Toggle Cell Code Visibility | - | `$(eye)` |

## Pluto.jl WebSocket プロトコル

### サーバー起動

```julia
import Pluto
Pluto.run(
    launch_browser=false,      # ブラウザを開かない
    host="127.0.0.1",
    port=1234,
    require_secret_for_access=false  # 開発用
)
```

### メッセージ形式

- **シリアライゼーション**: MessagePack (バイナリ)
- **ライブラリ**: `@msgpack/msgpack` (npm)

### 主要メッセージタイプ

| Client → Server | 説明 |
|-----------------|------|
| `connect` | 初期接続ハンドシェイク |
| `update_notebook` | セルコード・順序の更新 |
| `run_multiple_cells` | セル実行リクエスト |
| `interrupt_all` | 実行中断 |
| `reset_shared_state` | 状態リセット（全状態取得） |

| Server → Client | 説明 |
|-----------------|------|
| `👋` | 接続応答（セッション情報） |
| `notebook_diff` | 状態差分（JSONPatch形式） |
| `pong` | Ping応答 |

### セル実行フロー

```
Client                          Server
  │                               │
  │─── run_multiple_cells ───────▶│
  │    {cells: ["uuid1"]}         │
  │                               │ (リアクティブ実行)
  │◀─── notebook_diff ───────────│
  │    {running: true}            │
  │                               │
  │◀─── notebook_diff ───────────│
  │    {output: {...},            │
  │     running: false}           │
```

### 状態オブジェクト構造

```typescript
interface NotebookState {
    notebook_id: string;
    path: string;
    process_status: "ready" | "starting" | "waiting_for_permission";

    cell_inputs: {
        [cell_id: string]: {
            cell_id: string;
            code: string;
            code_folded: boolean;
        }
    };

    cell_results: {
        [cell_id: string]: {
            cell_id: string;
            output: any;           // レンダリング済み出力
            queued: boolean;
            running: boolean;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AtelierArith/better-pluto-client](https://github.com/AtelierArith/better-pluto-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
