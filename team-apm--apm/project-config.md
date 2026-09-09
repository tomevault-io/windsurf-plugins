---
trigger: always_on
description: AviUtl Package Manager (apm) — AviUtl のプラグイン・スクリプトを管理する Electron 製デスクトップアプリ。TypeScript + Vite (electron-forge)。UI は React + tRPC(main 窓は単一ルートの App、About 窓も React)。ビジネスロジックは main プロセス(src/main/services)にあり、preload はログ捕捉と tRPC bridge の公開のみ(初期化フローは renderer 側の startup.ts)。
---

# AGENTS.md

AviUtl Package Manager (apm) — AviUtl のプラグイン・スクリプトを管理する Electron 製デスクトップアプリ。TypeScript + Vite (electron-forge)。UI は React + tRPC(main 窓は単一ルートの App、About 窓も React)。ビジネスロジックは main プロセス(src/main/services)にあり、preload はログ捕捉と tRPC bridge の公開のみ(初期化フローは renderer 側の startup.ts)。

## 構成

| パス                 | 役割                                                                                                                                                 |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `src/main/`          | メインプロセス。`index.ts` = エントリ、`api/` = tRPC ルーター、`services/` = ビジネスロジック                                                        |
| `src/renderer/`      | 窓ごと(`main` / `about` / `splash`)。forge.config.ts の `renderer[]` と 1:1 対応。各ディレクトリの `index.html` がビルドの入口                       |
| `src/renderer/main/` | main 窓。単一 React ルート(`App.tsx`)+ タブごとのディレクトリ(`aviutl` / `packages` / `nicommons` / `settings` / `others`)+ 起動フロー(`startup.ts`) |
| `src/lib/`           | renderer から使う electron 依存モジュール(`ipcWrapper.ts` = preload 専用)                                                                            |
| `src/shared/`        | electron 非依存の純粋モジュール。ユニットテストの主対象                                                                                              |
| `src/common/ipc.ts`  | preload 専用 IPC のチャンネル名定義(他はすべて tRPC。理由はファイル内コメント)                                                                       |

プロセス構成・main プロセスの内訳・データフローの詳細は ARCHITECTURE.md を参照。

## コマンド

```
yarn lint       # prettier --check + eslint(報告のみ。自動修正は yarn fix)
yarn lint:ts    # tsc --noEmit
yarn test       # vitest run (src/**/*.test.ts)
yarn package    # electron-forge package(ThirdPartyNotices 生成込み)
yarn test:e2e   # Playwright E2E (e2e/)。パッケージ版を起動するため先に yarn package が必要。--user-data-dir で一時 userData を渡して起動するため実プロファイルは汚さない
yarn start      # 開発起動
```

PR 前に上記 3 つ(lint / lint:ts / test)がすべて緑であること。

## 確定方針(変更しない)

開発方針の単一ソース(旧 ROADMAP.md を統合)。変更は PR 経由で行う(履歴 = 意思決定ログ)。

- **apm は AviUtl1 専用。AviUtl2(ExEdit2)には対応しない**(#2163、2026-08 決定。理由は下の却下表)。ただしこの判断は 2026-08 時点の競争環境に依存しているため、**前提が崩れたら再評価する**(再評価条件: aviutl2-catalog の開発停止・apm-data 相当の資産が流用可能になる形式変更・インストーラ版のパス構造変更のいずれか)
- **v4 フルリライトはしない**。main 窓のタブ移行(Strangler Fig)は完了。新規画面・大改修する画面は React + tRPC で書く(About 窓が実装パターンの前例)
- **挙動変更と構造変更を同じ PR に混ぜない**。構造だけを変える PR では現行動作を保存し、途中で見つけたバグは issue 化して別 PR で直す
- v3.x を小出しリリース(v4 番号は使わない)。リリースは release-it(手順は RELEASING.md、ブランチ運用は BRANCHING.md)
- Windows メイン。明らかなクラッシュは直す。ビルドは 3 OS 継続 — mac/Linux 版に確たる実需要があるからではなく、開発環境が macOS であること(開発者自身の動作確認に必要)と、ファイル操作中心のアプリでクロスプラットフォーム維持コストが低いことが理由。維持コストが上がったら縮小を検討してよい
- dataURL は自由入力を維持(allowlist しない)。防御は `src/shared/resolvePath.ts` の同一オリジン + 親ディレクトリ脱出禁止 + 未承認オリジン追加時の一度だけの確認ダイアログ(#2377)+ README/SECURITY での注意喚起
- i18n は後回し(まず日本語のまま。英語 UI は #1879)
- メジャー依存更新は 1 PR = 1 major(dependabot は major を ignore — 下の「既知の固定」)
- テスト: ユニットは Vitest。electron に依存しない純粋関数を優先してテストする
- `src/shared/` = electron 完全非依存の純粋モジュール(main / renderer 両方から import 可)。electron に依存するものは `src/lib/`(renderer 系)や `src/main/` に置く

### 却下した選択肢(再提案しない)

| 選択肢                                 | 却下理由                                                                                                                                                                                                                                                                               |
| -------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| v4 全面リライト(#2169)                 | 機能喪失リスクが大きく、コンフリクトも解消不能に近い。Strangler Fig の段階移行で置き換えた                                                                                                                                                                                             |
| AviUtl2(ExEdit2)対応(#2163)            | プラグインが完全新形式で v1 と非互換のため apm-data の資産(285 件)が流用できない。installationPath 単一基準のパス解決がインストーラ版 AviUtl2(exe と ProgramData 分離・要管理者昇格)と構造的に不適合。専用マネージャの aviutl2-catalog が 1 年先行しており、後発参入の投資対効果がない |
| dataURL の allowlist 化                | 自作パッケージ・サードパーティデータの検証というユースケースを壊す                                                                                                                                                                                                                     |
| semantic-release / changesets への移行 | release-it + conventional-changelog で十分。ツール入れ替えのコストに見合わない                                                                                                                                                                                                         |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [team-apm/apm](https://github.com/team-apm/apm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
