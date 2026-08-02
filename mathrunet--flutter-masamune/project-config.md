---
trigger: always_on
description: 1. **日本語応答必須** → 全てのレスポンスは日本語で記述
---

# Masamuneフレームワーク開発ガイド v2.0

## 🚨 絶対原則（P0）

### 必ず守るべき鉄則
1. **日本語応答必須** → 全てのレスポンスは日本語で記述
2. **katana code必須** → 手動でのファイル作成は絶対禁止
3. **段階的バリデーション** → 1実装ごとに`flutter analyze && dart run custom_lint`実行
4. **全生成ファイルコミット** → `.m.dart`, `.g.dart`, `.freezed.dart`, テスト画像必須

### 開発フローの絶対順序
```
実装 → バリデーション → 修正 → 次の実装
```
この順序を絶対に崩さない。エラーは即座に対処。

## 🎯 開発フロー（P0）

### 基本サイクル
1. `katana code [type] [name]`でテンプレート生成
2. 実装を記述
3. `flutter analyze && dart run custom_lint`でバリデーション
4. エラーがあれば修正して再実行
5. UI変更時は`katana test update [ClassName]`
6. 次のコンポーネントへ

### 新規機能追加フロー
```bash
# 1. テンプレート生成
katana code page [PageName]          # ページ作成
katana code collection [Name]        # コレクション作成
katana code document [Name]          # ドキュメント作成
katana code controller [Name]        # コントローラー作成
katana code widget [Name]            # ウィジェット作成
katana code value [Name]             # フォーム値作成

# 2. コード生成（Freezed/JsonSerializable）
katana code generate

# 3. 実装とバリデーション（各コンポーネントごと）
flutter analyze && dart run custom_lint

# 4. UI更新時のテスト
katana test update [ClassName1],[ClassName2]
```

### Firebase Functions実装フロー
```bash
# 1. Functions生成
katana code server call [Name]      # 関数直接呼び出し
katana code server request [Name]   # HTTPリクエスト
katana code server schedule [Name]  # スケジューラー
katana code server firestore [Name] # Firestoreトリガー

# 2. サーバー側実装
# firebase/functions/src/[name].tsのprocessメソッド実装
# Node.js Masamuneパッケージ活用:
# - masamune_auth: 認証処理
# - masamune_firestore: データベース操作
# - masamune_notification: 通知送信
# - masamune_mail_sendgrid: メール送信
# - masamune_purchase_stripe: 決済処理
# - masamune_ai_openai: AI連携

# 3. クライアント側実装
# lib/functions/[name].dartでFunctionsAction/Response定義
final response = await appFunction.execute(
  TestFunctionsAction(
    companyId: "xxx",
    userId: "yyy",
  ),
);

# 4. デプロイ
katana deploy functions
```

### katana.yaml設定とkatana apply
```yaml
# katana.yaml例
name: myapp
package: com.example.myapp

# プラグイン設定（自動インストール）
firebase:
  enable: true
  firestore:
    enable: true
  auth:
    enable: true
    google: true
    apple: true
  functions:
    enable: true

# プラグイン例
purchase:
  enable: true
  type: stripe

# 適用コマンド（全設定を自動反映）
katana apply
```

### 完了前の必須作業
```bash
# 1. コードフォーマット
dart fix --apply lib && dart format . && flutter pub run import_sorter:main

# 2. バリデーション（エラー0必須）
flutter analyze && dart run custom_lint

# 3. UI更新時：ゴールデンテスト更新
katana test update [更新したクラス名]

# 4. 全体テスト実行
katana test debug

```

## 🛠️ コマンド早見表（P0）

### 頻出コマンド
| タスク | コマンド |
|--------|----------|
| **ページ作成** | `katana code page [name]` |
| **モデル作成** | `katana code collection/document [name]` |
| **コントローラー** | `katana code controller [name]` |
| **ウィジェット** | `katana code widget [name]` |
| **フォーム値** | `katana code value [name]` |
| **Functions作成** | `katana code server call/request/schedule/firestore [name]` |
| **コード生成** | `katana code generate` |
| **プラグイン適用** | `katana apply` |
| **バリデーション** | `flutter analyze && dart run custom_lint` |
| **フォーマット** | `dart fix --apply lib && dart format .` |
| **インポート整理** | `flutter pub run import_sorter:main` |
| **テスト更新** | `katana test update [class]` |
| **テスト実行** | `katana test debug` |
| **Functions deploy** | `katana deploy functions` |


## 🤖 エージェント選択ガイド（P1）

### 状況別エージェントマトリックス
| 状況 | エージェント | 目的 |
|------|-------------|------|
| **フレームワーク質問** | `masamune_framework_advisor` | 実装方法・ルール確認 |
| **パッケージ検討** | `package_advisor` | 最適パッケージ選定 |
| **UI実装** | `ui_builder` | デザインからコード生成 |
| **UIデバッグ** | `ui_debugger` | デザイン差分検出 |
| **テスト実行** | `test_runner` | テスト実行・分析 |
| **Firebase問題** | `firebase_flutter_debugger` | 連携問題解決 |

### 各エージェント概要
- **masamune_framework_advisor**: Model/Page/Controller/Widget/Form/プラグイン使用方法、ModelFieldValue活用
- **package_advisor**: Masamuneプラグイン確認、pub.dev検索、npm packages検討
- **ui_builder**: デザイン資産からUI実装、UniversalUI活用、レスポンシブ対応
- **ui_debugger**: 実装UIとデザイン比較、差分検出、修正提案
- **test_runner**: ゴールデンテスト更新、テスト実行、エラー解析
- **firebase_flutter_debugger**: Auth/Firestore/Functions連携デバッグ、ログ確認

※詳細は`~/.claude/agents/*.md`を参照

## 🔌 MCPサーバー活用ガイド（P1）

### MCPサーバー概要
Masamuneフレームワークでは、Claude CodeのMCP（Model Context Protocol）サーバーを積極的に活用して、
開発効率を大幅に向上させることを推奨します。以下のMCPサーバーを利用可能です。

### 利用可能なMCPサーバー

| サーバー | 主要機能 | 活用シーン |
|---------|---------|-----------|
| **mcp__dart** | pub検索, エラー解析, テスト実行 | パッケージ選定, デバッグ |
| **mcp__github** | Issue/PR, コード検索 | 実装例検索, PR管理 |
| **mcp__notion** | ページ操作, DB操作 | 仕様書参照, タスク管理 |
| **mcp__firebase** | プロジェクト管理, Crashlytics | セットアップ, エラー分析 |

### エージェント別MCPサーバー活用マトリックス

| エージェント | dart | github | notion | firebase | 主な用途 |
|-------------|------|--------|--------|----------|----------|
| **package_advisor** | ✓ | ✓ | - | - | pub.dev検索、類似実装検索 |
| **firebase_flutter_debugger** | ✓ | - | - | ✓ | エラー解析、ログ調査 |
| **masamune_framework_advisor** | ✓ | ✓ | ✓ | - | ドキュメント参照、実装例検索 |
| **ui_builder** | - | ✓ | ✓ | - | デザイン仕様参照、UI実装例検索 |
| **test_runner** | ✓ | - | - | - | テスト実行、エラー解析 |
| **ui_debugger** | ✓ | - | ✓ | - | ウィジェット解析、デザイン仕様確認 |

### MCPサーバー利用の注意点
- MCPツール優先使用（手動操作を避ける）
- 認証情報：GitHubは`secrets.yaml`、Firebaseは`firebase login`
- 必要最小限のAPI呼び出し、キャッシュ活用

### 📎 URL認識とMCPサーバー自動選択

URL提示時は以下の表に従ってMCPサーバーを選択：

| URL形式 | 使用MCPツール | 用途 |
|---------|--------------|-----|
| `notion.so/...` | `mcp__notion__API-retrieve-a-page` | ページ/仕様書取得 |
| `github.com/.../issues/` | `mcp__github__issue_read` | Issue内容取得 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mathrunet/flutter_masamune](https://github.com/mathrunet/flutter_masamune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
