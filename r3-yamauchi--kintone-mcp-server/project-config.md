---
trigger: always_on
description: - プロジェクトディレクトリ外のファイルやユーザーディレクトリを無断で操作・参照しない。
---

# プロジェクト向け AGENTS.md
## 基本方針
- 返答は必ず**日本語**で行うこと。
- プロジェクトディレクトリ外のファイルやユーザーディレクトリを無断で操作・参照しない。
- ホームディレクトリなどへのファイル作成は禁止。作業はこのリポジトリ内で完結させる。

## 実行環境
- **Node.js 20** 以降を使用する。
- 初回は `corepack enable` → `corepack prepare pnpm@10.13.1 --activate` → `pnpm install` の順で依存をインストールした後、`server.js` を起動して MCP Server を立ち上げる。
- 環境変数 `KINTONE_DOMAIN`, `KINTONE_USERNAME`, `KINTONE_PASSWORD` は必ずシェル等で設定し、`.env` ファイルへのフォールバックはサポートしない。

## 重要ドキュメント
- `docs/tool-annotations.md` — MCP仕様 2025-03-26 に対応したツールアノテーション一覧。
- `docs/implementation-status.md` — 未実装機能や拡張計画を記載。

## 1. プロジェクト概要


### 1.1 目的と役割

- このプロジェクトはkintoneとの連携を目的としたModel Context Protocol (MCP) サーバー
- Claude等のAIアシスタントからkintoneのデータにアクセス・操作するための橋渡し役
- kintoneとのやりとりは Node.js の標準 `fetch`/`FormData` API を直接使用

### 1.2 システム構成図

```mermaid
graph TD
    A[AIアシスタント] -->|MCP| B[kintone MCP Server]
    B -->|REST API| C[kintone]
    B -->|環境変数| D[認証情報]

    subgraph "kintone MCP Server"
    E[ツール層] --> F[サーバー層]
    F --> G[リポジトリ層]
    G --> H[モデル層]
    end
```

## 2. 基本方針

### 2.1 データ安全性の確保

- ユーザーデータを失うリスクのある機能はサポートしない
  - レコード、フィールド、アプリの削除機能は提供しない
  - 誤操作によるデータ損失を防止する

### 2.2 操作とインターフェース

- **読み取り・作成・更新操作**: 安全な操作のみを提供
- **シンプルなインターフェース**: 複雑な操作は避け、直感的に使えるツールを提供

### 2.3 機能範囲の制限

- **カスタマイズ機能非対応**:
  - kintoneのJavaScriptカスタマイズ機能には言及しない
  - CSSファイルによるカスタマイズ機能にも言及しない

### 2.4 用語の使用に関するルール

- kintoneのフィールドタイプ「SUBTABLE」は、日本語では「テーブル」と表現する
- メッセージ文言内などで「サブテーブル」という表現は避け、「SUBTABLE」または「テーブル」と表現する
- ユーザーが「サブテーブル」と表現した場合は、SUBTABLE（日本語では「テーブル」）のことと解釈してタスクを進める

## 3. アーキテクチャと設計

### 3.1 レイヤー構造

- **モデル層**: データモデルの定義
- **リポジトリ層**: kintone APIとの通信を担当
- **サーバー層**: MCPプロトコルの実装
- **ツール層**: 各種機能の実装

### 3.2 設計原則

- **依存関係**: 上位レイヤーから下位レイヤーへの一方向のみ
- **単一責任の原則**: 各クラスは明確に定義された単一の責任を持つ

### 3.3 ファイル分割の方針

- 機能ごとに適切にファイルを分割
- ファイルサイズが大きくなった場合は、内容や機能を分割することをユーザーに提案
- 関連する機能はディレクトリでグループ化

### 3.4 ディレクトリ構造

- **src/models**: データモデルの定義
- **src/repositories**: kintone APIとの通信を担当
- **src/server**: MCPサーバーの実装
- **src/server/tools**: 各種ツールの実装

## 4. 実装ガイドライン

### 4.1 非同期処理の原則

- **一貫性のある非同期パターン**: 関数内で非同期処理を行う場合は、必ず`async/await`または`Promise`を一貫して使用する
- **再帰呼び出しでの注意点**: 非同期関数内で自身を再帰的に呼び出す場合は、必ず`await`キーワードを使用する
- **同期/非同期の混在を避ける**: 同じ処理フローの中で同期処理と非同期処理を混在させない

```javascript
// 良い例
async function processRecords(records) {
  const results = [];
  for (const record of records) {
    const result = await processRecord(record);
    results.push(result);
  }
  return results;
}

// 避けるべき例（混在パターン）
function processRecords(records) {
  const results = [];
  for (const record of records) {
    processRecord(record).then(result => {
      results.push(result);
    });
  }
  return results; // 処理が完了する前に返却される
}
```

### 4.2 データ構造の整合性確保

- **複雑なオブジェクト構造の取り扱い**: 複雑なオブジェクト構造を操作する場合は、深いコピー（`JSON.parse(JSON.stringify())`など）を使用して参照の問題を避ける
- **Promiseオブジェクトの検出**: データ構造内にPromiseオブジェクトが混入していないか確認するデバッグログを出力する
- **入力データの検証**: `null`, `undefined`, 期待される型でない値など、異常なデータ構造に対する堅牢性を確保する

### 4.3 エラーハンドリングと堅牢性

- **詳細なエラーログ**: 問題の特定を容易にするため、処理の各段階で詳細なデバッグログを出力する
- **データ変換の透明性**: データ変換前後の状態をログに出力し、変換が正しく行われていることを確認できるようにする
- **自動修復機能**: 可能な場合は、異常なデータ構造を自動的に修復する機能を実装する（例：配列でない値を配列に変換）

```javascript
async function getRecord(appId, recordId) {
  try {
    if (!appId || !recordId) {
      throw new Error('アプリIDとレコードIDは必須です');
    }

    const response = await this.client.record.getRecord({
      app: appId,
      id: recordId
    });

    if (!response || !response.record) {
      throw new Error('レコードの取得に失敗しました');
    }

    return response.record;
  } catch (error) {
    console.error(`レコード取得エラー: ${error.message}`);
    throw new McpError(
      ErrorCode.InternalError,
      `レコードの取得に失敗しました: ${error.message}`
    );
  }
}
```

### 4.4 ツール実装の方針

- 各ツールは独立して実装し、相互依存を避ける
- 入力パラメータの検証を徹底

### 4.5 自動補正機能

- **フィールドコードの自動生成**: ラベルから適切なコードを生成
- **計算フィールドの式**: formulaプロパティが指定された場合、expressionプロパティに自動変換
- **桁区切り表示**: digit=trueの場合、formatをNUMBER_DIGITに自動設定
- **単位記号の位置**: 単位記号に基づいて適切な位置を自動判定
- **数値フィールドのdisplayScale**: 空文字列の場合は自動的に削除

### 4.6 kintone API連携

- Node.js 標準の fetch と FormData を直接使用
- 認証情報は環境変数から取得し、ハードコードしない

## 5. 機能仕様と制限事項

### 5.1 kintoneの計算フィールドの仕様

- **kintone標準の計算機能のみをサポート**:
  - kintoneがサポートする関数と演算子のみを使用
  - Excel/Spreadsheetなどで使用できる関数の多くはサポート外

- **自動補正機能**:
  - formulaプロパティが指定された場合: expressionプロパティに自動変換
  - digit=trueの場合: formatをNUMBER_DIGITに自動設定

- **サポートされていない関数と代替方法**:
  - サポートされていない関数を検出した場合:
    - 自動的に変換や補正は一切行わない
    - 代替方法を提案するエラーメッセージを表示するのみ
    - ユーザーが自分で正しい関数や式に修正する必要がある
  - テーブル名.フィールド名パターンや構文エラーも同様に自動修正せず、ガイダンスのみ提供
    - 例：「Table1.field1」のような参照は自動変換せず、正しい参照方法を提案する

### 5.2 特殊機能

- **単位記号の位置自動判定**:
  - 通貨記号（¥, $, €など）はBEFORE（数値の前）に配置
  - 単位記号（kg, cm, 個など）はAFTER（数値の後）に配置
  - これらの基本ルールに基づいて自動判定を行う
- **フィールドコードの自動生成**: ラベルから適切なコードを生成
- **フォームレイアウトのサイズ指定**: 数値形式のみ受け付け、単位指定は無視

### 5.3 制限事項

- **大容量ファイル制限**:
  - 現状、1MB以上のファイルのダウンロードはサポートできていない
  - MCPプロトコルの制約によるもので、将来的にサポートする方法を検討する
- **システムフィールド制限**:
  - システムフィールド（CREATOR, MODIFIER等）を新規追加することはできない
  - 既存のシステムフィールドは名前変更等の一部操作が可能

### 5.4 ルックアップフィールドの実装方針

- **ルックアップフィールドの基本構造**:
  - ルックアップフィールドは基本的なフィールドタイプ（SINGLE_LINE_TEXT、NUMBERなど）に、lookup属性を追加したものである
  - フィールドタイプとして "LOOKUP" を指定するのではなく、適切な基本タイプを指定し、その中にlookupプロパティを設定する

- **ツールの使用方法**:
  - `create_lookup_field`ツールは設定オブジェクトを生成するだけのヘルパーツールである
  - 実際にフィールドを追加するには、この結果を`add_fields`ツールに渡す必要がある
  - ツールの結果には注意書きと使用例を含める


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [r3-yamauchi/kintone-mcp-server](https://github.com/r3-yamauchi/kintone-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
