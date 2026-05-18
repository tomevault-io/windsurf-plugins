---
trigger: always_on
description: nomad-cssは、フレームワーク非依存のプレーンなCSSライブラリです。以下の要件を満たすために開発されています：
---

# nomad-css Library - Copilot Instructions

## プロジェクト概要

nomad-cssは、フレームワーク非依存のプレーンなCSSライブラリです。以下の要件を満たすために開発されています：

- **マルチフレームワーク対応**: Node.js、ASP.NET Core、PHPなど様々なフレームワークで利用可能
- **低依存性**: TailwindやBootstrapのようなライブラリへの高い依存を避ける
- **レガシーブラウザ対応**: 古いブラウザでも動作する（OKLCHなどの新技術を避ける）
- **テーマ切り替え可能**: 複数のテーマを切り替えられる設計
- **カスタマイズの制限**: テーマ確定後は大きなカスタマイズを制限

## コンポーネント実装の方針

このプロジェクトは特定のテーマに沿った実装を行うのではなく、**各テーマの実装に必要なプロパティ（CSS変数）を準備すること**が目的です。

### 基本原則

- **構造的な機能に注力**: コンポーネントの構造やレイアウトに関する機能を実装する
- **汎用的なバリエーション**: filled, tonal, outlined, text などの一般的なスタイルパターンを提供する
- **セマンティックカラーはテーマ層の責務**: nomad-css-ui層でprimary, secondary等のセマンティックバリエーションは定義しない
- **拡張ポイントの提供**: テーマ層でカスタマイズできるようCSS変数で制御可能にする

### コンポーネントの実装例

#### ボタン

```html
<!-- 構造的なバリエーション（nomad-css-ui で定義） -->
<button class="button filled">Filled</button>
<button class="button tonal">Tonal</button>
<button class="button outlined">Outlined</button>
<button class="button text">Text</button>

<!-- アイコン付きボタン -->
<button class="button"><span class="icon">★</span> With Icon</button>
<button class="button icon-only"><span class="icon">★</span></button>

<!-- セマンティックカラー（テーマ層で定義） -->
<button class="button primary">Primary</button>
```

### 実装すべき内容

| コンポーネント | nomad-css-ui層で実装する内容                               | テーマ層で実装する内容           |
| -------------- | ---------------------------------------------------------- | -------------------------------- |
| Button         | filled, tonal, outlined, text, アイコン配置, サイズ        | セマンティックカラー, 角丸, 影   |
| Input          | サイズ, 状態（disabled, readonly）, バリエーション変数定義 | ボーダースタイル, フォーカス効果 |
| Card           | レイアウト構造, elevation用CSS変数                         | シャドウ, ボーダースタイル       |
| Table          | 構造, striped, bordered, hover                             | カラーバリエーション             |

## アーキテクチャ

### 階層構造

```
┌─────────────────────────────────────┐
│           テーマ層                   │  src/themes/<theme-name>/
│   (fluent2, material3, etc.)        │  CSS変数を上書きして外観を定義
├─────────────────────────────────────┤
│       nomad-css-ui 層               │  src/nomad-css-ui/
│   (button, form-control, table...)  │  プレーンなコンポーネント定義
├─────────────────────────────────────┤
│       ユーティリティ層               │  src/utilities/
│   (palette, border, fonts...)       │  テーマ非依存のユーティリティ
└─────────────────────────────────────┘
```

### ディレクトリ構造

```
src/
├── utilities/              # ユーティリティ層（テーマ・フレームワーク非依存）
│   ├── _prefixes.scss      # CSS変数グローバルプレフィックス定義
│   ├── _palette.scss       # パレット生成 Mixin (generate-palette)
│   ├── _border.scss        # ボーダーユーティリティクラス
│   ├── _breakpoints.scss   # レスポンシブBreakpoint Mixin
│   ├── _container.scss     # レイアウトユーティリティクラス (display, flex等)
│   ├── _fonts.scss         # 見出しフォントサイズ/ウェイト変数とクラス
│   ├── _shadows.scss       # シャドウ変数とユーティリティクラス (shadow-0〜5)
│   └── _spacing.scss       # マージン・パディング・ギャップクラス (m-/p-/gap-)
│
├── nomad-css-ui/           # コンポーネント層（utilitiesに依存）
│   ├── _nomad-css-ui.scss  # エントリーポイント
│   ├── components/         # 各コンポーネント（25種）
│   │   ├── _components.scss    # コンポーネントのエントリーポイント
│   │   ├── _button.scss        # ボタン
│   │   ├── _form-control.scss  # テキスト入力
│   │   ├── _form-hint.scss     # フォームヒント
│   │   ├── _select.scss        # セレクトボックス
│   │   ├── _checkbox.scss      # チェックボックス
│   │   ├── _radio.scss         # ラジオボタン
│   │   ├── _range.scss         # レンジスライダー
│   │   ├── _progress.scss      # プログレスバー
│   │   ├── _switch.scss        # スイッチ
│   │   ├── _table.scss         # テーブル
│   │   ├── _alert.scss         # アラート
│   │   ├── _badge.scss         # バッジ
│   │   ├── _chip.scss          # チップ
│   │   ├── _spinner.scss       # スピナー
│   │   ├── _skeleton.scss      # スケルトン
│   │   ├── _accordion.scss     # アコーディオン
│   │   ├── _tooltip.scss       # ツールチップ
│   │   ├── _breadcrumb.scss    # パンくずリスト
│   │   ├── _pagination.scss    # ページネーション
│   │   ├── _card.scss          # カード
│   │   ├── _divider.scss       # 区切り線
│   │   ├── _dialog.scss        # ダイアログ
│   │   ├── _avatar.scss        # アバター
│   │   └── _list.scss          # リスト
│   └── utilities/          # nomad-css-ui 内部ユーティリティ
│       ├── _palette.scss       # パレット統合（HUE + セマンティック）
│       ├── _color-defines.scss # 共通カラー変数 (border, text, surface)
│       ├── _color-utils.scss   # テキスト/背景カラーユーティリティクラス
│       ├── _hue-colors.scss    # HSL色相環 12色定義
│       ├── _semantics.scss     # セマンティックカラー定義
│       ├── _fonts.scss         # nomad-css-ui フォント変数
│       └── _border.scss        # パレットカラーのボーダー拡張
│
└── themes/                 # テーマ層（nomad-css-ui の CSS変数を上書き）
    ├── default/            # デフォルトテーマ（最小限の上書き）
    ├── fluent2/            # Microsoft Fluent 2 風
    ├── material3/          # Google Material 3 風
    ├── material2/          # Google Material 2 風
    ├── material1/          # Google Material 1 風
    ├── ant/                # Ant Design 風
    ├── apple-hig/          # Apple HIG 風
    ├── bootstrap/          # Bootstrap 風
    ├── carbon/             # IBM Carbon 風
    ├── chakra/             # Chakra UI 風
    ├── daisyui/            # DaisyUI 風
    ├── primer/             # GitHub Primer 風
    ├── shadcn/             # shadcn/ui 風
    ├── spectrum/           # Adobe Spectrum 風
    ├── aqua/               # macOS Aqua 風
    ├── holo/               # Android Holo 風
    ├── win95/              # Windows 95 風
    ├── win7/               # Windows 7 風

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Uta-member/nomad-css](https://github.com/Uta-member/nomad-css) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
