---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

ComfyUI用プロンプトワード記録Chrome拡張機能（Manifest V3）。エヴァンゲリオン初号機テーマのUI。階層的なグループ構造でワードを管理し、重複排除した最終プロンプトを生成する。選択組み合わせをメタデータ付きプリセットとして保存・還元できる。総括欄には文字列変換ルールを適用でき、元ワード本文は変えずに表示・コピー内容だけを変換できる。

## 開発コマンド

```bash
# 開発サーバ起動（ブラウザで UI 確認）
npm run dev
# → http://localhost:5173/src/popup.html

# 本番ビルド（型チェック + ビルド）
npm run build
# → dist/ に拡張機能を出力

# リント
npm run lint

# テスト（watch）
npm test
# テスト（1回実行）
npm run test:run
# テスト（カバレッジ）
npm run test:coverage
```

## Chrome拡張機能の読み込み

1. `npm run build` 実行
2. Chrome で `chrome://extensions` を開く
3. 「デベロッパー モード」を有効化
4. 「パッケージ化されていない拡張機能を読み込む」→ `dist/` フォルダを選択

## 技術スタック

- **フレームワーク**: React 19 + Vite + TypeScript
- **スタイル**: Tailwind CSS 4 (エヴァンゲリオン初号機カラーテーマ)
- **アニメーション**: Motion (framer-motion の軽量版)
- **アイコン**: React Icons
- **状態管理**: React Context API（永続状態と画面限定の一時状態を分離）
- **永続化**: chrome.storage.local
- **拡張機能ビルド**: @crxjs/vite-plugin
- **テスト**: Vitest（`src/lib` の純粋関数ユニットテスト）

**注意**: React Compiler (babel-plugin-react-compiler) は未使用。

## アーキテクチャ

### 状態管理の中核

**[src/context/PromptContext.tsx](src/context/PromptContext.tsx)** がグローバル状態を管理：
- `RootState`: ルートグループ配列 + プリセット + 変換ルール
- 全ての更新操作は immutable（structuredClone ベース）
- debounce（220ms）で chrome.storage.local へ自動保存
- 選択ワードの収集・変換ルール適用・重複排除・差分計算は useMemo で派生
- プリセット関連: `savePreset` / `applyPreset` / `updatePresetMeta` / `updatePresetEntries` / `analyzePresetApply` / `diffPresetEntries` など
- 変換ルール関連: `addRule` / `updateRule` / `deleteRule` / `setRuleEnabled` / `reorderRules`

### データモデル ([src/types.ts](src/types.ts))

```typescript
// ROOT_VERSION = 2（rules 追加後）

RootState {
  version: number
  rootGroups: Group[]
  presets?: PromptPreset[]
  rules: PromptTransformRule[]  // 常に配列（旧データは正規化で []）
}

Group {
  id, name, collapsed
  groups: Group[]    // 無制限ネスト可能
  words: Word[]
}

Word {
  id, text, note, selected
  strength?: number  // 0..10（0=デフォルト / 1=() / 2..10=(text:1.x)）
  image?: string     // Base64 data URL（最大420×420）
}

// プリセット（選択組み合わせ + 生成メタ）
PromptPreset {
  id, name
  baseModel, baseModelKind
  loras?: PresetModelRef[]        // { model, strength }
  controlNets?: PresetModelRef[]
  metadata: PresetMetadata        // steps, cfg, sampler, scheduler, width, height
  image: string                   // プレビュー画像（最大560px JPEG data URL）
  description?: string
  entries: PresetEntry[]          // { wordId, text, strength } text は差分通知用
  createdAt, updatedAt?
}

// 総括欄プロンプトの文字列変換ルール
// 元ワード本文は変更せず、表示・コピー内容だけを変換
PromptTransformRule {
  id, name
  from: string   // 変換前（リテラル。正規表現ではない）
  to: string     // 変換後（空文字可＝削除ルール）
  enabled: boolean  // 新規作成時は必ず false
}

PresetFormData {
  // 新規保存・メタ編集の入力（id/createdAt 以外）
  // metadata は未入力時 optional。保存時に正規化して metadata へ落とす
}
```

### ツリー操作 ([src/lib/tree/](src/lib/tree/))

単一責任の原則（SRP）に基づき、機能ごとにモジュール分割された純粋関数群。すべての更新は immutable（structuredClone ベース）。

**モジュール構成**:

- **[tree/id.ts](src/lib/tree/id.ts)** (18行): ID生成
  - `genId()`: ユニークID生成（タイムスタンプ + カウンタ + ランダム）

- **[tree/factory.ts](src/lib/tree/factory.ts)**: オブジェクト生成
  - `createWord()`, `createGroup()`: 新規オブジェクト生成

- **[tree/search.ts](src/lib/tree/search.ts)** (42行): ツリー検索
  - `findGroup()`: グループをIDで検索
  - `isDescendant()`: 子孫関係判定（循環参照防止）

- **[tree/searchHits.ts](src/lib/tree/searchHits.ts)**: 検索ヒット収集（UI フィルタ用）
  - `wordMatchesQuery()`: ワード本文（normalizeText）・注釈の部分一致
  - `collectSearchHits()`: DFS で直下ヒットがあるグループだけを収集（グループ名は対象外）

- **[tree/immutable.ts](src/lib/tree/immutable.ts)** (25行): immutable更新ヘルパ
  - `clone()`: structuredCloneによる深いコピー
  - `mutateGroup()`: グループを安全に更新

- **[tree/group.ts](src/lib/tree/group.ts)** (161行): グループ操作
  - `addGroup()`, `renameGroup()`, `deleteGroup()`
  - `toggleCollapse()`, `setCollapsed()`
  - `moveGroup()`: ドラッグ&ドロップ対応の複雑な移動ロジック（循環検出、アンカーID基準で挿入位置決定）
  - `GroupDropTarget`: 移動先の型定義（into / before / after / root）

- **[tree/word.ts](src/lib/tree/word.ts)** (87行): ワード操作
  - `addWord()`, `updateWord()`, `deleteWord()`
  - `toggleWord()`, `setWordSelected()`, `setWordStrength()`
  - `reorderWords()`: 同一グループ内の並替（HTML5 DnD / Motion layout 対応）
  - `moveWord()`, `findDuplicateWords()`

- **[tree/collector.ts](src/lib/tree/collector.ts)** (57行): 選択ワード収集
  - `collectSelected()`: 深さ優先で選択ワードを収集（出現順維持）
  - `groupHasSelection()`: 選択ワード存在チェック（折り畳み徽章用）
  - `countSelectedWords()`, `countSelectedWordsInGroup()`
  - `SelectedWordRef`: 選択ワード参照の型定義

- **[tree/navigation.ts](src/lib/tree/navigation.ts)** (71行): グループ列挙・展開
  - `collectAllGroups()`: 全グループを平坦化（時計ロードマップ用）
  - `expandGroupPath()`: 指定グループとその祖先を展開
  - `GroupRef`: グループ参照の型定義

- **[tree/preset.ts](src/lib/tree/preset.ts)** (370行): プリセット操作
  - `collectPresetEntries()`: 現在の選択から PresetEntry 配列を構築
  - `savePreset(form)`: 現在の選択 + フォーム情報を新規保存（同名でも上書きしない。重複名チェックはフォーム側）
  - `updatePresetMeta(id, form)`: メタ情報のみ更新（entries は維持）
  - `updatePresetEntries(id)`: ワード情報だけを現在の選択で更新
  - `applyPreset(id)`: 全ワードを未選択にし、entries の wordId で selected/strength を当てはめる。プリセット外ワードの強度は維持する（text は復元しない）
  - `analyzePresetApply(id)`: 還元前に id 欠落・text 変更を検査
  - `diffPresetEntries(id)`: 現在の選択 vs プリセット entries の差分（追加/削除/強度変更/text変更）
  - `deletePreset()`, `renamePreset()`, `reorderPresets()`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Gladiale/ComfyUI_Prompt_Recorder_EVA-01-Style](https://github.com/Gladiale/ComfyUI_Prompt_Recorder_EVA-01-Style) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
