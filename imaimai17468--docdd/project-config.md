---
trigger: always_on
description: このファイルは、Cursorを使用した開発ワークフローの標準手順を定義します。
---

# Development Workflow Rules

このファイルは、Cursorを使用した開発ワークフローの標準手順を定義します。
新機能の追加やバグ修正を行う際は、以下のフローに従ってください。

## 基本方針

- **できるだけ全てのフェーズを実行する**（タイプ別の推奨フローは下記参照）
- **各フェーズで TodoWrite ツールを活用**して進捗を管理する
- **不明点があれば AskUserQuestion で確認**してから進める
- **エラーが発生したら必ず修正**してから次のフェーズに進む
- **コミット前にすべてのチェックがパス**していることを確認
- **段階的にコミット**し、大きすぎる変更を避ける
- **Next.js MCP、Chrome DevTools MCPを積極的に活用**して動作確認を徹底する

---

## クイックリファレンス

変更のタイプに応じて、適切なフローを選択してください：

| 変更タイプ | 推奨フロー | 所要時間目安 | 説明 |
|-----------|-----------|-------------|------|
| **新機能追加** | Phase 1-11 全て | 60-120分 | 完全なワークフロー |
| **中規模バグ修正** | 1,4,5,6,8,9A,10,11 | 30-60分 | 調査→実装→テスト→確認 |
| **UI/デザイン調整** | 1,3,4,5,8,9A,10,11 | 20-40分 | UIデザインレビュー含む |
| **小規模リファクタ** | 1,4,5,8,10,11 | 15-30分 | 既存パターン踏襲 |
| **タイポ修正** | 5,8,10,11 | 5分 | 設定ファイルや小さな修正 |
| **ドキュメント更新** | 5,10,11 | 5-10分 | ドキュメントのみの変更 |

**Phase 9について:**
- **Phase 9A（簡易確認）**: 必須 - Next.js MCPでエラーチェック
- **Phase 9B（詳細検証）**: 任意 - Chrome DevToolsでの詳細確認

---

## フェーズ概要

### 必須フェーズ vs 任意フェーズ

#### 必須フェーズ（ほぼすべてのケースで実行）
1. **Phase 1: Investigation & Research** - Context7/Kiriで調査
4. **Phase 4: Planning** - TodoWriteで計画立案
5. **Phase 5: Implementation** - Serenaでコード実装
8. **Phase 8: Quality Checks** - bun run でチェック実行
9. **Phase 9A: Runtime Verification** - Next.js MCPで動作確認
10. **Phase 10: Git Commit** - コミット作成
11. **Phase 11: Push** - リモートへプッシュ

#### 状況に応じて実行（推奨）
2. **Phase 2: Architecture Design** - 新機能や大規模変更時
3. **Phase 3: UI/UX Design** - UI変更がある場合
6. **Phase 6: Testing & Stories** - ロジック変更がある場合
7. **Phase 7: Code Review** - リファクタリングが必要な場合
9. **Phase 9B: Browser Verification** - 詳細な動作確認が必要な場合

---

## 使用エージェント/コマンド

以下のカスタムコマンドが利用可能です：

- **`component-refactoring-specialist`** (`.cursor/commands/app-code-specialist.md`) - Reactコンポーネントのリファクタリング専門家。ロジック抽出、プレゼンターパターン適用、ディレクトリ構造の再編成を担当
- **`test-guideline-enforcer`** (`.cursor/commands/test-guideline-enforcer.md`) - Vitest / React Testing Libraryを使用したテストコードの品質、構造、命名規約を強制
- **`storybook-story-creator`** (`.cursor/commands/storybook-story-creator.md`) - プロジェクトルールに準拠したStorybookストーリーの作成とメンテナンス
- **`ui-design-advisor`** (`.cursor/commands/ui-design-advisor.md`) - ダークテーマに焦点を当てたUI/UXデザイン専門家。レイアウトのレビューと改善提案を担当
- **`spec-document-creator`** (`.cursor/commands/spec-document-creator.md`) - 拡張可能な仕様書作成コマンド。機能仕様、API仕様、アーキテクチャ仕様など複数のドキュメントタイプをサポート
- **`adr-memory-manager`** (`.cursor/commands/adr-memory-manager.md`) - AI用のADR（Architecture Decision Record）を自動記録・検索・管理。JSON形式で機械可読性を最優先に設計
- **`project-onboarding`** (`.cursor/commands/project-onboarding.md`) - プロジェクトの構造、ドメイン知識、技術スタック、アーキテクチャパターンを分析・記録。新規プロジェクトのオンボーディングに最適

---

## Workflow Steps

### Phase 1: Investigation & Research (調査フェーズ) 【必須】

**使用ツール**: Context7 MCP, Kiri MCP

#### 1. 既存コードベースの調査（Kiri MCPを使用）

Kiri MCPはSerenaより高度な検索機能を提供します。セマンティック検索、フレーズ認識、依存関係分析などを活用してください。

**1-1. コンテキスト自動取得（推奨）**
```
mcp__kiri__context_bundle
goal: 'user authentication, login flow, JWT validation'
limit: 10
compact: true
```
- タスクに関連するコードスニペットを自動でランク付けして取得
- `goal`には具体的なキーワードを使用（抽象的な動詞は避ける）
- `compact: true`でトークン消費を95%削減

**1-2. 具体的なキーワード検索**
```
mcp__kiri__files_search
query: 'validateToken'
lang: 'typescript'
path_prefix: 'src/auth/'
```
- 関数名、クラス名、エラーメッセージなど具体的な識別子で検索
- 広範な調査には`context_bundle`を使用

**1-3. 依存関係の調査**
```
mcp__kiri__deps_closure
path: 'src/auth/login.ts'
direction: 'inbound'
max_depth: 3
```
- 影響範囲分析（inbound）や依存チェーン（outbound）を取得
- リファクタリング時の影響調査に最適

**1-4. コードの詳細取得**
```
mcp__kiri__snippets_get
path: 'src/auth/login.ts'
```
- ファイルパスがわかっている場合に使用
- シンボル境界を認識して適切なセクションを抽出

#### 2. ライブラリドキュメントの確認
- Context7 MCPを使用して最新のライブラリドキュメントを取得
- Next.js, React, その他使用するライブラリの最新情報を確認
- `mcp__context7__resolve-library-id` → `mcp__context7__get-library-docs` の順で実行

#### 3. 既存決定の確認（ADR参照）【必須】

**⚠️ 重要: このステップは必ず実行すること**
- **コード調査だけでは不十分**: `codebase_search`やKiri MCPで既存パターンを確認しても、ADR確認は別途必須
- **実装前に必ず確認**: 既存のアーキテクチャ決定に従うか、新しい決定が必要かを判断
- **ADR確認方法**:
  1. `docs/adr/index.json`を確認して関連ADRを特定
  2. 関連するADRファイル（`docs/adr/decisions/*.json`）を読み込む
  3. 特に以下のカテゴリのADRを確認:
     - **アーキテクチャパターン**: コンポーネントの設計パターン、データ取得戦略、状態管理方法など
     - **ドメイン知識**: プロジェクト固有のドメインエンティティ、ビジネスロジック、必須実装要件など
       - **重要**: ドメイン知識ADRに新規コンポーネント作成時の必須実装要件が記載されている場合、必ず確認
       - `affected_files`や`affected_components`を確認して、必要な登録や設定ファイルへの追加を特定
     - **プロジェクト構造**: ディレクトリ構造、命名規則、ファイル配置ルールなど
  4. **新規コンポーネント作成時の確認事項**:
     - デモコンポーネントやドキュメント実装が必要か確認
     - レジストリや設定ファイルへの登録が必要か確認
     - 必要なメタデータや設定の追加が必要か確認
  5. 実装がADRの決定と一致しているか確認
  6. 新しい決定が必要な場合は`adr-memory-manager`エージェントを使用して記録

**ADR確認のタイミング:**
- Phase 1で初回確認（必須）
- Phase 4（Planning）の前に再確認（推奨）
- Phase 5（Implementation）の前に最終確認（推奨）

#### 4. 調査結果の整理
- 既存パターンやコーディング規約を把握
- 再利用可能なコンポーネントやユーティリティを特定
- Kiriで取得したコンテキストを基に実装方針を決定
- **既存ADRと照合して決定の一貫性を確認**（必須）

**完了チェックリスト:**
- [ ] Kiri MCPで関連コードを特定
- [ ] 必要なライブラリのドキュメントを確認
- [ ] 既存パターンと依存関係を把握
- [ ] **ADRを確認し、既存決定を理解**（必須 - コード調査とは別に実行）
- [ ] **ドメイン知識ADRを確認し、新規コンポーネント作成時の必須実装要件を判断**（新規コンポーネント作成時は必須）
- [ ] 実装がADRの決定と一致していることを確認

---

### Phase 2: Architecture Design (アーキテクチャ設計) 【推奨：新機能/大規模変更時】

**使用エージェント**: component-refactoring-specialist, spec-document-creator, adr-memory-manager

**このフェーズをスキップできるケース:**
- 既存パターンに完全に倣う場合
- 1ファイル以内の小さな修正
- ドキュメントやスタイルのみの変更

#### 1. 技術的方針の決定
- ファイル配置、ディレクトリ構造の決定
- 状態管理の方法（useState, useContext, 外部ライブラリなど）
- データフローとコンポーネント間の関係性の設計
- APIエンドポイントやデータ取得戦略の決定

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imaimai17468/docdd](https://github.com/imaimai17468/docdd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
