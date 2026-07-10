---
trigger: always_on
description: このドキュメントは、mcp-uiを使用したインタラクティブチャットデモプロジェクトの設計書です。
---

# MCP-UI Demo Project

このドキュメントは、mcp-uiを使用したインタラクティブチャットデモプロジェクトの設計書です。

## プロジェクト概要

**目的**: Model Context Protocol (MCP) の UI拡張仕様である mcp-ui を使用して、チャットUI内にインタラクティブなフォームや選択UIを埋め込むデモアプリケーションを構築する。

**コンセプト**:
- ユーザーは通常のチャットUIでLLMと対話
- LLMが必要に応じてMCPツールを呼び出し、UIResource（フォーム・選択肢など）を取得
- UIResourceがチャットメッセージ内に埋め込まれて表示
- ユーザーがUIを操作すると、その結果がLLMにフィードバックされ、会話が継続

## アーキテクチャ

### システムフロー

```
┌─────────────┐
│   User      │
└──────┬──────┘
       │ テキスト入力
       ▼
┌─────────────────────────┐
│   Chat UI (React)       │
│  - メッセージ表示       │
│  - UIResourceRenderer   │
└──────┬──────────────────┘
       │ HTTP/WS
       ▼
┌─────────────────────────┐
│  Express Server         │
│  - API endpoints        │
│  - OpenAI integration   │
└──────┬──────────────────┘
       │ OpenAI API
       ▼
┌─────────────────────────┐
│  OpenAI Chat API        │
│  - Function calling     │
└──────┬──────────────────┘
       │ Tool call
       ▼
┌─────────────────────────┐
│  MCP Server             │
│  - @mcp-ui/server       │
│  - UIResource generation│
└──────┬──────────────────┘
       │ UIResource
       ▼
┌─────────────────────────┐
│  UIResourceRenderer     │
│  - Sandboxed iframe     │
│  - postMessage protocol │
└──────┬──────────────────┘
       │ User action
       ▼
     Feedback to OpenAI
```

### データフロー

1. **ユーザー入力 → LLM**
   - ユーザーがチャットにメッセージを入力
   - ExpressサーバーがOpenAI Chat Completions APIにリクエスト

2. **LLM → MCP Server**
   - LLMがfunction callingでMCPツールを呼び出し
   - MCP Serverが適切なUIResourceを生成して返却

3. **UIResource → クライアント**
   - UIResource（HTMLまたはRemote DOM）がクライアントに送信
   - `UIResourceRenderer`がiframe内でUIを描画

4. **ユーザー操作 → フィードバック**
   - ユーザーがUI（フォーム・ボタンなど）を操作
   - `postMessage`でアクションがホストに送信
   - ExpressサーバーがOpenAI APIに操作結果をフィードバック

## 技術スタック

### Frontend
- **React 18**: UIフレームワーク
- **TypeScript**: 型安全性
- **Vite**: 高速ビルドツール
- **@mcp-ui/client**: UIResourceのレンダリング

### Backend
- **Node.js 18+**: ランタイム
- **Express**: Webフレームワーク
- **TypeScript**: 型安全性
- **OpenAI SDK**: Chat Completions API (function calling)

### MCP Server
- **@mcp-ui/server**: UIResource生成SDK
- **TypeScript**: 実装言語

### 通信
- **HTTP/REST**: 基本的なAPI通信
- **WebSocket** (オプション): リアルタイム更新
- **postMessage**: iframe間通信

## ディレクトリ構成

```
jsconf-mcp-ui-demo/
├── CLAUDE.md                 # このファイル
├── README.md                 # プロジェクト概要・セットアップ手順
├── package.json              # ルートpackage.json (workspace設定)
│
├── client/                   # フロントエンド (React + Vite)
│   ├── src/
│   │   ├── components/
│   │   │   ├── ChatUI.tsx           # メインチャットUI
│   │   │   ├── MessageList.tsx      # メッセージ表示
│   │   │   ├── MessageItem.tsx      # 個別メッセージ
│   │   │   ├── UIResourceMessage.tsx # UIResourceを含むメッセージ
│   │   │   └── InputArea.tsx        # テキスト入力エリア
│   │   ├── hooks/
│   │   │   └── useChat.ts           # チャットロジック
│   │   ├── types/
│   │   │   └── chat.ts              # 型定義
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── package.json
│   ├── tsconfig.json
│   └── vite.config.ts
│
├── server/                   # バックエンド (Express)
│   ├── src/
│   │   ├── index.ts                 # エントリーポイント
│   │   ├── routes/
│   │   │   └── chat.ts              # チャットAPI
│   │   ├── services/
│   │   │   ├── openai.ts            # OpenAI統合
│   │   │   └── mcpClient.ts         # MCPクライアント
│   │   ├── types/
│   │   │   └── index.ts             # 型定義
│   │   └── config/
│   │       └── index.ts             # 設定管理
│   ├── package.json
│   └── tsconfig.json
│
├── mcp-server/               # MCPサーバー
│   ├── src/
│   │   ├── index.ts                 # MCPサーバーエントリー
│   │   ├── tools/
│   │   │   ├── formTool.ts          # フォームUI生成ツール
│   │   │   └── choiceTool.ts        # 選択肢UI生成ツール
│   │   └── types/
│   │       └── index.ts             # 型定義
│   ├── package.json
│   └── tsconfig.json
│
└── shared/                   # 共通型定義・ユーティリティ
    ├── src/
    │   └── types/
    │       └── mcp.ts               # MCP関連の共通型
    ├── package.json
    └── tsconfig.json
```

## 実装するユースケース

### 1. フォーム入力

**シナリオ**: ユーザーがLLMに情報を提供する際、テキストで入力する代わりにフォームUIを表示

**例**:
```
User: 「レストランを予約したいです」
LLM: 「予約情報を入力してください」
→ フォームUI表示（名前、日時、人数、連絡先）
User: フォームに入力して送信
LLM: 「〇〇様、△月△日△時に□名での予約を承りました」
```

**実装**:
- MCP Tool: `show_reservation_form`
- UIResource: `text/html` 形式のフォーム
- フィールド: テキスト入力、日時ピッカー、数値入力
- 送信時: `postMessage` でツール呼び出し `submit_reservation`

### 2. 選択肢の提示

**シナリオ**: ユーザーが複数の選択肢から選ぶ際、ボタンやラジオボタンで視覚的に提示

**例**:
```
User: 「おすすめの料理を教えて」
LLM: 「どのジャンルがお好みですか？」
→ 選択肢UI表示（和食・洋食・中華・イタリアン）
User: 「和食」を選択
LLM: 「和食のおすすめは...」
```

**実装**:
- MCP Tool: `show_choices`
- UIResource: `text/html` 形式のボタングループまたはラジオボタン
- 選択時: `postMessage` でツール呼び出し `select_option`

## コア実装詳細

### OpenAI Function Calling

```typescript
// Function定義
const functions = [
  {
    name: 'show_reservation_form',
    description: 'ユーザーにレストラン予約フォームを表示する',
    parameters: {
      type: 'object',
      properties: {
        restaurantName: { type: 'string' }
      }
    }
  },
  {
    name: 'submit_reservation',
    description: '予約情報を送信する',
    parameters: {
      type: 'object',
      properties: {
        name: { type: 'string' },
        date: { type: 'string' },
        time: { type: 'string' },
        partySize: { type: 'number' },
        contact: { type: 'string' }
      },
      required: ['name', 'date', 'time', 'partySize', 'contact']
    }
  }
];


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arashigaoka/jsconf-mcp-ui-demo](https://github.com/arashigaoka/jsconf-mcp-ui-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-10 -->
