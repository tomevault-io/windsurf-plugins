---
trigger: always_on
description: **APG Patterns Examples** は、WAI-ARIA Authoring Practices Guide (APG) のコンポーネントパターンを React、Vue、Svelte、Astro の4つのフレームワークで実装し、実際に動作するデモと詳細なコード解説を提供するプロジェクトです。
---

# CLAUDE.md - APG Patterns Examples

## プロジェクト概要

**APG Patterns Examples** は、WAI-ARIA Authoring Practices Guide (APG) のコンポーネントパターンを React、Vue、Svelte、Astro の4つのフレームワークで実装し、実際に動作するデモと詳細なコード解説を提供するプロジェクトです。

### プロジェクトの目標

- APG準拠のアクセシブルなコンポーネント実装
- フレームワーク別のベストプラクティス提供
- プロダクション品質のコード
- 日英バイリンガル対応
- GitHub Pages での自動デプロイ

---

## 技術スタック（Astro リビルド版）

| レイヤー       | 技術                           |
| -------------- | ------------------------------ |
| フレームワーク | Astro (Islands アーキテクチャ) |
| コンテンツ     | MDX                            |
| デモ           | React / Vue / Svelte / Astro   |
| スタイリング   | Tailwind CSS + shadcn/ui       |
| コード表示     | Shiki                          |
| フォーマット   | Prettier + @takazudo/mdx-formatter |
| テスト         | Vitest + Playwright            |
| 検索           | Pagefind                       |
| 多言語         | Astro i18n                     |
| デプロイ       | GitHub Pages                   |

---

## ディレクトリ構成

```
src/
├── components/                # サイト UI (shadcn/ui)
│   └── ui/
├── lib/                       # ユーティリティ
│   ├── frameworks.ts          # フレームワーク定義の一元管理
│   └── pattern-meta-types.ts  # PatternMeta 型定義
├── patterns/                  # APG パターン実装
│   └── button/                # 例: button パターン
│       ├── meta.ts            # パターンメタデータ（単一の真実源）
│       ├── DemoSection.astro  # 全フレームワーク統合デモ
│       ├── TestingDocs.astro  # テストドキュメント
│       ├── Button.tsx         # React 実装
│       ├── Button.vue         # Vue 実装
│       ├── Button.svelte      # Svelte 実装
│       ├── Button.astro       # Astro 実装（Web Components）
│       ├── ButtonDemo.tsx     # React デモラッパー（必要時）
│       ├── Button.test.tsx    # テスト
│       └── button.md          # AI向け定義ファイル（llm.md）
├── content/
│   └── accessibility-docs/    # アクセシビリティ解説（MDX）
│       └── button/
│           ├── en.mdx         # 英語
│           └── ja.mdx         # 日本語
├── layouts/
├── pages/
│   ├── patterns/
│   │   ├── [pattern]/
│   │   │   └── [framework]/
│   │   │       └── index.astro  # 動的ルーティング（en）
│   │   └── index.astro          # パターン一覧
│   ├── ja/patterns/
│   │   └── [pattern]/
│   │       └── [framework]/
│   │           └── index.astro  # 動的ルーティング（ja）
│   └── ...
├── i18n/
└── styles/
```

### データ駆動アーキテクチャ

各パターンの `meta.ts` が単一の真実源（Single Source of Truth）となる。1つの `meta.ts` から 4フレームワーク × 2言語 = 8ページが動的に生成される。

- **`meta.ts`**: タイトル、説明、TOC、リソース、フレームワーク別メタデータ（ソースファイル、API Props/Events/Slots）を `PatternMeta` 型で定義。全テキストは `Record<Locale, string>` で i18n 対応
- **`DemoSection.astro`**: 4フレームワークの実装を静的 import し、`framework` prop で切り替えて表示
- **動的ルーティング**: `[pattern]/[framework]/index.astro` が `import.meta.glob()` で全パターンの `meta.ts` を検出し、`getStaticPaths()` でビルド時にページ生成

**パスエイリアス**（tsconfig.json）:

- `@/*` → `./src/*`
- `@patterns/*` → `./src/patterns/*`

---

## 内部ドキュメント

| ファイル                                                             | 内容                                            | 参照タイミング |
| -------------------------------------------------------------------- | ----------------------------------------------- | -------------- |
| [CODING_RULES.md](CODING_RULES.md)                                   | コーディングスタイル規約                        | コード実装時   |
| [.internal/site-specification.md](.internal/site-specification.md)   | サイト仕様書（技術選定、URL設計、実装方針）     | 実装時         |
| [.internal/testing-strategy.md](.internal/testing-strategy.md)       | テスト設計方針（DAMP原則、APG準拠テストの観点） | テスト実装時   |
| [.internal/architecture-review.md](.internal/architecture-review.md) | アーキテクチャレビュー（現状課題、改善選択肢）  | 参考資料       |
| [.internal/llm-md-template.md](.internal/llm-md-template.md)         | AI向け定義ファイル（llm.md）のテンプレート      | パターン追加時 |

---

## 開発ガイド

### コンポーネント設計原則

#### 1. HTML属性継承パターン

**React**:

```typescript
export interface ToggleButtonProps extends Omit<
  React.ButtonHTMLAttributes<HTMLButtonElement>,
  'onClick' | 'type' | 'aria-pressed'
> {
  initialPressed?: boolean;
  onToggle?: (pressed: boolean) => void;
}
```

**Vue**:

```vue
<script setup>
defineOptions({ inheritAttrs: false })
</script>
<template>
  <button v-bind="$attrs">
</template>
```

**Svelte**:

```svelte
<button {...$$restProps}>
```

**Astro** (Web Components):

```astro
<apg-toggle-button>
  <button type="button" aria-pressed={initialPressed}>
    <slot />
  </button>
</apg-toggle-button>

<script>
  class ApgToggleButton extends HTMLElement {
    connectedCallback() {
      // イベントリスナー設定
    }
  }
  customElements.define('apg-toggle-button', ApgToggleButton);
</script>
```

#### 2. アクセシビリティファースト

- `aria-pressed` 等の ARIA 属性による状態管理
- キーボードナビゲーション（Space/Enter）
- スクリーンリーダー対応
- フォーカス管理

#### 3. AccessibilityDocs 構成

各パターンのアクセシビリティ解説は `src/content/accessibility-docs/{pattern}/{en,ja}.mdx` に MDX で記述する。`TestingDocs.astro` がこれを読み込んで表示する。以下のセクション構成で作成する:

1. **Native HTML Considerations** (該当パターンのみ)
   - ネイティブ HTML 要素の推奨（例: `<a>`, `<table>`, `<input type="number">`）
   - カスタム実装が必要になるユースケース
   - ネイティブ vs カスタムの判断基準

2. **WAI-ARIA Roles**
   - 使用する role の一覧と説明
   - 各 role の適用対象要素

3. **WAI-ARIA States / Properties**
   - `aria-*` 属性の詳細（値、必須/任意、変更トリガー）
   - 仕様へのリンク

4. **Keyboard Support**
   - キーボード操作の一覧表
   - 各キーのアクション説明

**ネイティブ HTML セクションが必要なパターン例:**

- Link → `<a href>`
- Table → `<table>`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [masuP9/apg-patterns-examples](https://github.com/masuP9/apg-patterns-examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
