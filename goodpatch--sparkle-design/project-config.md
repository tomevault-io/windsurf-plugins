---
trigger: always_on
description: Testing guidelines following t_wada best practices
---


# Sparkle Design Component Testing Instructions

## Overview
このプロジェクトは Next.js + TypeScript + Vitest + Testing Library を使用した React コンポーネントライブラリです。すべてのコンポーネントに対して **t_wada さんのベストプラクティス**に従った堅牢なテストを作成・維持します。

## Testing Philosophy (t_wada Best Practices)
このプロジェクトでは、t_wada さんが提唱するテスト駆動開発とテスト設計のベストプラクティスを厳密に遵守します：

- **Intention-revealing**: テストの意図が明確に分かるテスト名と構造
- **Granular**: 細かい粒度でのテスト分割
- **Property-based**: コンポーネントのプロパティベースのテスト
- **Accessibility**: アクセシビリティの確認
- **Error/Edge cases**: エラーケースとエッジケースのカバレッジ
- **Maintainable**: 保守しやすく、リファクタリングに強いテスト
- **Reliable**: フレーキーでない、安定したテスト

## AI/Agent Testing Workflow
AI/エージェントがテスト結果を分析・修正する際は、以下のワークフローに従います：

### 1. 中間ログファイルへの出力
テスト実行結果は必ず中間ログファイルに出力してからAIが分析します：

```bash
# テスト実行結果をログファイルに出力
pnpm test > test-output.log 2>&1

# AI分析用の現在のテスト状況確認
pnpm test 2>&1 | tee current-test-status.log

# 特定の失敗したテストのみ抽出
grep -A 5 -B 5 "FAIL\|✗\|Error" test-output.log > test-failures.log
```

### 2. ログファイル分析パターン
AIは以下のパターンでログファイルを分析します：

```bash
# 失敗テスト数の確認
tail -20 test-output.log | grep -E "failed|passed|total"

# エラー詳細の抽出
grep -E "expect|received|AssertionError" test-output.log

# TypeScriptエラーの確認
grep -A 3 "TS[0-9]" test-output.log
```

### 3. ログファイルクリーンアップ
分析完了後は中間ログファイルを削除：

```bash
rm -f test-output.log current-test-status.log test-failures.log test-final.log
```

## Project Structure
```
src/
├── components/ui/
│   ├── [component]/
│   │   ├── index.tsx          # Component implementation
│   │   └── index.test.tsx     # Component tests
├── test/
│   └── helpers.ts             # Shared test helpers
```

## Test File Structure
各コンポーネントのテストファイルは以下の構造に従う：

```tsx
/**
 * @jest-environment jsdom
 */

import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest'
import { TestContainer, EventHelpers, A11yHelpers, StyleHelpers } from '@/test/helpers'
import { ComponentName } from './index'

describe('ComponentName', () => {
  let testContainer: TestContainer

  beforeEach(() => {
    testContainer = new TestContainer()
    testContainer.setup()
  })

  afterEach(() => {
    testContainer.cleanup()
  })

  describe('Basic Rendering', () => {
    // 基本的なレンダリングテスト
  })

  describe('Variant Styling', () => {
    // バリアントスタイリングテスト
  })

  describe('User Interaction', () => {
    // ユーザーインタラクションテスト
  })

  describe('Accessibility', () => {
    // アクセシビリティテスト
  })

  describe('Edge Cases', () => {
    // エッジケーステスト
  })
})
```

### Required Test Environment Setup
各テストファイルで必須の設定：

1. **JSDoc environment directive**: `/** @jest-environment jsdom */`
2. **TestContainer setup/cleanup**: コンポーネントをレンダリングするテストでは`beforeEach`/`afterEach`でのライフサイクル管理を行う（`it.todo`のみのファイルは省略可）
3. **Helper imports**: 必要に応じて`StyleHelpers`, `EventHelpers`, `A11yHelpers`をimport

## Shared Test Helpers

### TestContainer
レンダリングとDOMクエリ用のヘルパークラス：
```tsx
testContainer.render(<Component />)
testContainer.queryInput()    // input要素を取得
testContainer.queryButton()   // button要素を取得
testContainer.getContainer()  // container要素を取得
```

### EventHelpers
イベント発火用のヘルパー：
```tsx
EventHelpers.click(element)        // クリックイベント
EventHelpers.change(input, value)  // 入力変更イベント
EventHelpers.keyDown(element, key) // キーダウンイベント
EventHelpers.focus(element)        // フォーカスイベント
```

### A11yHelpers
アクセシビリティチェック用のヘルパー：
```tsx
A11yHelpers.hasAriaLabel(element, label)
A11yHelpers.hasRole(element, role)
A11yHelpers.isDisabled(element)
```

### StyleHelpers
CSSクラステスト用のヘルパー：
```tsx
StyleHelpers.hasClass(element, className)           // 単一クラスの確認
StyleHelpers.hasClasses(element, classNames)        // 複数クラスの確認
StyleHelpers.getComputedStyleProperty(element, prop) // 計算済みスタイルの取得
```

### Test Fix Best Practices (from usage insights)
テスト修正時は以下のベストプラクティスに従う：

1. **共有ヘルパーを優先する**: テストを修正する際は、一時的なワークアラウンドやメソッドオーバーライドではなく、上記の共有テストヘルパー（`TestContainer`, `EventHelpers`, `A11yHelpers`, `StyleHelpers`）を使用する
2. **既存パターンを確認する**: 新しいテストユーティリティやフィクスチャを追加する前に、コードベースに同様のパターンが既に存在しないか確認する
3. **再利用可能なインフラを好む**: 応急処置的な修正よりも、再利用可能なテストインフラを構築する

❌ **Wrong** - 一時的なワークアラウンド
```tsx
// 特定のテストだけで使う一時的なモック
const mockFunction = vi.fn().mockImplementation(() => 'test')
```

✅ **Correct** - 共有ヘルパーを使用
```tsx
// 共有ヘルパーを使用してイベントを発火
EventHelpers.click(button)
EventHelpers.change(input, 'new value')
```

## Component-Specific Guidelines

### Regular Components (Button, Input, Badge, etc.)
- 基本レンダリング
- バリアント（size, variant, status等）
- ユーザーインタラクション
- アクセシビリティ
- エラーハンドリング
- エッジケース

### Portal-based Components (Dialog, Modal, Select)
Portal コンポーネントは jsdom での直接テストが難しい場合があります。安定している場合は最小限のスモークテストを追加し、不安定な場合は`it.todo`で理由を残します。

**スモークテスト例（安定時）**
```tsx
describe('DialogComponent', () => {
  it('opens and closes via trigger', async () => {
    // minimal smoke test
  })
})
```

**`it.todo` 例（不安定時）**
```tsx
describe('DialogComponent', () => {
  // Portal-based components are challenging to test with jsdom
  // due to portal rendering behavior and DOM limitations.

  it.todo('should render dialog with trigger button')
  it.todo('should open dialog when trigger is clicked')
  // ... その他のtodo
})
```

### Controlled Components
```tsx
it('supports controlled mode', () => {
  const handleChange = vi.fn()
  testContainer.render(<Component value="test" onChange={handleChange} />)
  const input = testContainer.queryInput()

  EventHelpers.change(input, 'new value')

  expect(handleChange).toHaveBeenCalled()

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [goodpatch/sparkle-design](https://github.com/goodpatch/sparkle-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
