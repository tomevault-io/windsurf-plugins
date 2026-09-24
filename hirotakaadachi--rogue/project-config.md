---
trigger: always_on
description: Turn-based roguelike in a single HTML/CSS/JS stack. Canvas rendering + Web Audio API (synthesized sounds, no external sound libs).
---

# Minimal Rogue RPG — AI Modification Guide

## Overview
Turn-based roguelike in a single HTML/CSS/JS stack. Canvas rendering + Web Audio API (synthesized sounds, no external sound libs).
- **Goal:** Descend to B100F, destroy the Dungeon Core
- **Post-clear:** "Deep" mode (B101F+), endless with second ending possible
- **Map:** 25 rows × 40 columns, tile-based

## File Structure
```
index.html   — canvas, HUD elements, loads script.js (update `?v=N` on release)
style.css    — layout only (dark theme, no game logic)
script.js    — everything: data, logic, rendering (~38000+ lines)
bgm_*.mp3    — background music tracks (HTML Audio)
```

**Workflow tip:** this is a single-page HTML file — just open `index.html` in a browser to test. No build step. Bump `?v=NN` in the script tag when you want to bust the browser cache after editing.

---

## ⚠️ ABSOLUTE RULE — Enemy hostile color is ALWAYS `#f87171`

**When setting an enemy's color for a hostile/aggressive state, ALWAYS use `#f87171`. Never use `#ef4444` or any other red variant.**

- The standard hostile enemy color in this game is `#f87171` (see the LETTER type fallback in `draw()`)
- `#ef4444` is a brighter/different red and has been incorrectly used multiple times — do NOT use it for enemy `eColor`
- This applies to every new enemy type, every special enemy state, every `eColor =` assignment for hostile enemies, forever
- Damage text numbers and `spawnFloatingText` colors are separate and unaffected by this rule
- `_justAngered` temporary flash is also a separate effect and unaffected
- If you are about to write `eColor = '#ef4444'` — stop. Change it to `#f87171`

---

## ⚠️ ABSOLUTE RULE — addLog() is English-only

**`addLog()` must NEVER contain Japanese text. English only, always, without exception.**

- The bottom-left log panel is English-only by design. Japanese display capability does not exist there and will never be added.
- This applies to every `addLog()` call, in every situation, forever.
- Story/narrative text in Japanese → use `showStoryPages()` (dialogue window) instead.
- Item/action feedback in Japanese → use `spawnFloatingText()` instead.
- If you are about to write `addLog("...日本語...")` — stop. Rewrite it in English or use a different display method.

---

## ⚠️ 実装前チェックリスト（修正のたびに必ず確認）

### 1. 世界観・仕様を尊重する
- **グリッドに沿わない動きは絶対に入れない。** タイル座標（整数）以外での移動・描画をしない。
- **キャラクターサイズを統一する。** 既存敵より大きい・小さいフォントサイズを勝手に使わない。
- **仕様にないものを勝手に追加しない。** 新しいレンダリング方式・座標システム・エフェクトは明示的に許可を得てから追加。
- **新機能は「すでにある素材の組み合わせ」として実装する。** 既存の敵タイプ・タイル・システムを流用できないか先に検討する。

### 2. 大きな実装を始める前に必ず確認する
- **ゴールイメージを言語化してもらう：**「どんな見た目・動作を想定していますか？」
- **使う素材を確認する：**「既存の敵・タイル・システムを使いますか？」
- **世界観との整合性を確認する：**「他のどの部屋・敵に近いイメージですか？」
- 選択肢（AskUserQuestion）を使って確認してから実装に入る。

### 3. バグ報告を受けたら症状を先に聞く
- コードを先に読み始めない。まず「具体的にどんな操作でどうなりましたか？」と聞く。
- 現象を把握してから原因を絞り込む。

---

## コード内ナビゲーション早見表

### セクションジャンプ（grep）
すべてのセクション一覧：`grep -n "===== SECTION" script.js`

| セクション | grep コマンド |
|-----------|-------------|
| 定数・Canvas設定 | `grep -n "SECTION: CONSTANTS"` |
| 敵タイプセット | `grep -n "SECTION: ENEMY TYPE"` |
| 指輪データ | `grep -n "SECTION: RINGS DATA"` |
| ゲーム変数 | `grep -n "SECTION: GAME OBJECT"` |
| サウンド | `grep -n "SECTION: SOUND EFFECTS"` |
| BGM | `grep -n "SECTION: BGM"` |
| ゲーム状態フラグ | `grep -n "SECTION: GAME STATE"` |
| マップ・プレイヤー・敵データ | `grep -n "SECTION: CORE GAME DATA"` |
| セーブ/ロード | `grep -n "SECTION: SAVE"` |
| UI更新 | `grep -n "SECTION: UI UPDATES"` |
| マップ生成 | `grep -n "SECTION: MAP GENERATION"` |
| タイトル/ゲームオーバー描画 | `grep -n "SECTION: TITLE"` |
| メニュー/ショップ/ステータス描画 | `grep -n "SECTION: MENU"` |
| メイン draw() 関数 | `grep -n "SECTION: MAIN DRAW"` |
| ブロック設置 | `grep -n "SECTION: BLOCK PLACEMENT"` |
| フロア13スクロール壁 | `grep -n "SECTION: SCROLL WALL"` |
| プレイヤーアクション | `grep -n "SECTION: PLAYER ACTION"` |
| 妖精システム | `grep -n "SECTION: FAIRY"` |
| 狂人システム | `grep -n "SECTION: MADMEN"` |
| 敵死亡処理 | `grep -n "SECTION: ENEMY FALL"` |
| 敵ターン処理 | `grep -n "SECTION: ENEMY TURN"` |
| レーザーシステム | `grep -n "SECTION: LASER"` |
| 入力処理 | `grep -n "SECTION: INPUT"` |

### 特定フロアのコードへジャンプ
`initMap()` 内の各フロアブロック先頭に `// ----- FLOOR N:` マーカーがある：
```
grep -n "FLOOR 50:"  script.js   → 50Fのコードへ
grep -n "FLOOR 100:" script.js   → 100Fのコードへ
```

主要フロアは専用関数に抽出済み（`_initFloor50()` など）。
関数一覧：`grep -n "^function _initFloor"  script.js`

### 特定の敵AIブロックへジャンプ
`grep -n "// ===== AI: TYPE_NAME"` で `enemyTurn()` 内の各敵AIブロック先頭に飛べる。

### ENEMY_DEFS（敵ベーススタッツ参照テーブル）
`grep -n "^const ENEMY_DEFS"` で定義箇所へ。
ランダムスポーン cascade で使用。固定フロアの特殊スポーンは直接記述のまま。

---

## Key Constants (script.js top)
```js
TILE_SIZE = 20    // pixels per tile
ROWS = 25         // map height
COLS = 40         // map width
```

## SYMBOLS (tile types)
```js
SYMBOLS.WALL      '#'   SYMBOLS.FLOOR     '.'
SYMBOLS.STAIRS    '>'   SYMBOLS.DOOR      '+'
SYMBOLS.ICE       '~'   SYMBOLS.LAVA      '^'
SYMBOLS.POISON    '%'   SYMBOLS.GRASS     ','
SYMBOLS.BLOCK     'B'   SYMBOLS.CORE      'C'   // Boss core (100F)
SYMBOLS.MERCHANT  'M'   SYMBOLS.FIRE_BLOCK 'F'
```

---

## 用語・名称対照表

ユーザーとの会話で使われる通称と、コード内の正式名の対照表。
実際のシンボル記号は `script.js` の `const SYMBOLS = { ... }` (line 35) を参照。

> **敵指定のルール:** ユーザーが敵を指定する際は、**画面上に表示されるアイコン（アルファベット）を優先する**。内部コード名ではなく見た目の文字で指示が来るため、アイコン列を先に照合すること。例：「B を配置して」→ BOAR（`B`）であり、BLAZE（内部名）ではない。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HirotakaAdachi/Rogue](https://github.com/HirotakaAdachi/Rogue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
