---
trigger: always_on
description: - Target: Obsidian Community Plugin (TypeScript → bundled JavaScript).
---

# Obsidian community plugin

## Project overview

- Target: Obsidian Community Plugin (TypeScript → bundled JavaScript).
- Entry point: `main.ts` compiled to `main.js` and loaded by Obsidian.
- Required release artifacts: `main.js`, `manifest.json`, and optional `styles.css`.

## Project rules
- プラグインのソースコードは、/Users/hiroyaiizuka/Desktop/Evergreens/.obsidian/plugins/taskchute-plus/srcに作成にあります。
- コードを実装したら、npm run testと、npm run lint、npm run buildを実行して、エラーがないことを確認してください。
- TaskChute Plusのコード変更後は、完了報告前に必ず`obsidian-e2e-tester`スキルで実機E2Eを実施し、PASSを確認すること（詳細: /Users/hiroyaiizuka/Desktop/Evergreens/.agents/skills/obsidian-e2e-tester/SKILL.md）。
- 要件定義や仕様書は、/Users/hiroyaiizuka/Desktop/Evergreens/.obsidian/plugins/taskchute-plus/.kiro/steeringに作成してください。
- 一時的に記載するドキュメントや実装のチェックリストは、/Users/hiroyaiizuka/Desktop/Evergreens/.obsidian/plugins/taskchute-plus/tmpに作成してください。
- メモリーで記載するノートについては、/Users/hiroyaiizuka/Desktop/Evergreens/.obsidian/plugins/taskchute-plus/memoryに作成してください。


## 基本方針
- 不明な点は積極的に質問する
- 質問する時は常にAskUserQuestionを使って回答させる
- **選択肢にはそれぞれ、推奨度と理由を提示する**
  - 推奨度は⭐の5段階評価


## Basic Memory ワークフロー

メモリの読み書き・検索・構造の詳細は **memory-manager スキル** に集約。

- **セッション開始時**: `memories/corrections/lessons.md` を読む


### メモリ構造

```
memory/
├── schemas/           # スキーマ定義（触らない）
├── events/            # 実装ログ・機能追加
├── bugfixes/          # バグ調査と修正
├── investigations/    # アーキテクチャ探索・原因調査
├── designs/           # 設計決定・ADR
├── corrections/       # 失敗と教訓の蒸留
│   ├── inbox.md       # ミスをすぐ書く
│   ├── lessons.md     # 蒸留された教訓
│   └── graduated.md   # 仕組み化済み
├── reviews/           # コードレビュー所見
└── archive/           # 古いメモ
```

### 記録ルール

| 何をした | カテゴリ | 保存先 | type |
|---------|----------|--------|------|
| 機能実装・リファクタリング | event | `events/` | event |
| バグ修正 | bugfix | `bugfixes/` | bugfix |
| コード調査・分析 | investigation | `investigations/` | investigation |
| 設計決定・技術選定 | design | `designs/` | design |
| コードレビュー | review | `reviews/` | review |
| ミス・失敗 | correction | `corrections/inbox.md` に追記 | correction |

### 検索
```bash
/Users/hiroyaiizuka/.local/bin/bm tool search-notes "{検索語}" --project taskchute-plus-memory
```
## Environment & tooling

- Node.js: use current LTS (Node 18+ recommended).
- **Package manager: npm** (required for this sample - `package.json` defines npm scripts and dependencies).
- **Bundler: esbuild** (required for this sample - `esbuild.config.mjs` and build scripts depend on it). Alternative bundlers like Rollup or webpack are acceptable for other projects if they bundle all external dependencies into `main.js`.
- Types: `obsidian` type definitions.


## File & folder conventions

- **Organize code into multiple files**: Split functionality across separate modules rather than putting everything in `main.ts`.
- Source lives in `src/`. Keep `main.ts` small and focused on plugin lifecycle (loading, unloading, registering commands).
- **Example file structure**:
  ```
  src/
    main.ts           # Plugin entry point, lifecycle management
    settings.ts       # Settings interface and defaults
    commands/         # Command implementations
      command1.ts
      command2.ts
    ui/              # UI components, modals, views
      modal.ts
      view.ts
    utils/           # Utility functions, helpers
      helpers.ts
      constants.ts
    types.ts         # TypeScript interfaces and types
  ```
- **Do not commit build artifacts**: Never commit `node_modules/`, `main.js`, or other generated files to version control.
- Keep the plugin small. Avoid large dependencies. Prefer browser-compatible packages.
- Generated output should be placed at the plugin root or `dist/` depending on your build setup. Release artifacts must end up at the top level of the plugin folder in the vault (`main.js`, `manifest.json`, `styles.css`).

## Manifest rules (`manifest.json`)

- Must include (non-exhaustive):  
  - `id` (plugin ID; for local dev it should match the folder name)  
  - `name`  
  - `version` (Semantic Versioning `x.y.z`)  
  - `minAppVersion`  
  - `description`  
  - `isDesktopOnly` (boolean)  
  - Optional: `author`, `authorUrl`, `fundingUrl` (string or map)
- Never change `id` after release. Treat it as stable API.
- Keep `minAppVersion` accurate when using newer APIs.
- Canonical requirements are coded here: https://github.com/obsidianmd/obsidian-releases/blob/master/.github/workflows/validate-plugin-entry.yml


## Coding conventions

- TypeScript with `"strict": true` preferred.
- **Keep `main.ts` minimal**: Focus only on plugin lifecycle (onload, onunload, addCommand calls). Delegate all feature logic to separate modules.
- **Split large files**: If any file exceeds ~200-300 lines, consider breaking it into smaller, focused modules.
- **Use clear module boundaries**: Each file should have a single, well-defined responsibility.
- Bundle everything into `main.js` (no unbundled runtime deps).
- Avoid Node/Electron APIs if you want mobile compatibility; set `isDesktopOnly` accordingly.
- Prefer `async/await` over promise chains; handle errors gracefully.


## Agent do/don't

**Do**
- Add commands with stable IDs (don't rename once released).
- Provide defaults and validation in settings.
- Write idempotent code paths so reload/unload doesn't leak listeners or intervals.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hiroyaiizuka/taskchute-for-obsidian](https://github.com/hiroyaiizuka/taskchute-for-obsidian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
