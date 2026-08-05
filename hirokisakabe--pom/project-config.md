---
trigger: always_on
description: pom (PowerPoint Object Model) — TypeScript library for declaratively describing PowerPoint presentations. Calculates Flexbox-style layouts with yoga-layout and generates PPTX files with `@pptx-glimpse/document`.
---

# AGENTS.md

pom (PowerPoint Object Model) — TypeScript library for declaratively describing PowerPoint presentations. Calculates Flexbox-style layouts with yoga-layout and generates PPTX files with `@pptx-glimpse/document`.

## Agent Instructions の配置方針

- AI agent 向け作業ルールの**正本は各階層の `AGENTS.md`**（このファイルおよび `packages/*/AGENTS.md`）。Codex など `AGENTS.md` を読む agent はこれを直接参照する。
- Claude Code 向けには各 `AGENTS.md` と同階層に `CLAUDE.md`（実ファイル）を置き、内容は原則 `@AGENTS.md` import のみとする。symlink は Windows 互換性と GitHub 上での可読性のため使わない。
- Claude 固有のルールが将来必要になった場合のみ、`CLAUDE.md` の `@AGENTS.md` の下に Claude 専用セクションを追加する。
- 配置一覧:
  - ルート `AGENTS.md` — リポジトリ全体の共通ルールとリリースフロー（このファイル）
  - `packages/pom/AGENTS.md` — コアライブラリのルール（Feature Addition Checklist / Preview Workflow / Text Measurement）
  - `packages/pom-cli/AGENTS.md` / `packages/pom-editor/AGENTS.md` / `packages/pom-jsx/AGENTS.md` / `packages/pom-md/AGENTS.md` / `packages/pom-vscode/AGENTS.md` — 各パッケージ別ルール

## Tech Stack

TypeScript 5.x, yoga-layout 3.2.1, @pptx-glimpse/document 0.12.0, opentype.js 1.3.x, fast-xml-parser 5.x, zod 4.x, Vitest, ESLint, Prettier, pnpm workspace

## Behavioral Principles

- Read existing code before making changes — especially check the 3-stage pipeline impact scope
- When adding features, follow the Feature Addition Checklist in `packages/pom/AGENTS.md`
- VRT baseline updates must use Docker environment (`pnpm run vrt:docker:update`)
- When changes span multiple packages, explicitly state the impact scope

## Commands (from `packages/pom/`)

```bash
pnpm run build           # TypeScript compilation
pnpm run lint            # ESLint
pnpm run lint:deps       # Dependency layer boundary check (dependency-cruiser)
pnpm run fmt             # Prettier formatting
pnpm run typecheck       # Type checking
pnpm run knip            # Detect unused code
pnpm run test:run        # Run tests
pnpm run vrt:docker:update  # Update VRT baseline (Docker)
```

Root: `pnpm --filter @hirokisakabe/pom run <script>`

## Directory Structure

```
packages/
├── pom/              # Core library — src/ (parseXml/ → calcYogaLayout/ → toPositioned/ → renderPptx/), vrt/, preview/, docs/, main.ts
├── pom-cli/          # CLI tool — preview and build presentations
├── pom-editor/       # React component for visual DnD AST editing — PomAstEditor
├── pom-jsx/          # JSX/TSX authoring package
├── pom-md/           # Markdown → pom XML converter
├── pom-vscode/       # VS Code extension for live preview
apps/
└── website/          # Documentation website (Next.js), content → pom/docs symlink
```

## Architecture

PPTX generation pipeline: **calcYogaLayout** → **toPositioned** → **renderPptx**. Additionally, **autoFit** adjusts slides when content overflows.

Existing PPTX reading and structural round-trip work should treat `@pptx-glimpse/document` as the first candidate dependency (#895 option 2: pom → `@pptx-glimpse/document` one-way dependency). If typed model coverage is insufficient, prefer its `packageGraph.rawParts` OOXML escape hatch before adding new ZIP/XML handling in pom.

### Public API (`@hirokisakabe/pom`)

- `buildPptx(xml, slideSize, options?)` — XML string → PPTX
- `BuildPptxResult`, `ParseXmlError`, `DiagnosticsError`, `Diagnostic`, `DiagnosticCode`
- `WritablePptx` — `buildPptx()` の出力 facade (`write` / `writeFile` / `stream`)
- `TextMeasurementMode` (`"opentype"` | `"fallback"` | `"auto"`), `FontInput` (`ArrayBuffer` / `Uint8Array` の layout measurement 用 font data), `SlideMasterOptions`
- `extractThemeTokensFromPptx(buffer)` — PPTX bytes → `ThemeTokens[]` (`slideMaster` 配下の表示 layout 順に、text / background / primary / secondary / accent3..6 を 6 桁大文字 hex で返す)
- `ThemeTokens`, `FALLBACK_THEME_TOKENS`
- `extractSlideMastersAsPptx(buffer)` — PPTX bytes → PPTX bytes (`Promise<ArrayBuffer>`)。各 slideMaster 配下の表示 layout ごとに空白スライド 1 枚だけを持つ PPTX に変換する。列挙順は `extractThemeTokensFromPptx` と同一なので、両者の出力配列を zip してスライドとテーマをペアにできる
- `parseXml(xml)` — XML string → `POMNode[]` (PascalCase tags, Zod-validated attributes). トップレベル `<Theme>` でデザイントークン（配色）を宣言でき、色属性の `$name` 参照は parse 時に解決される（`<Theme>` 自体はノードにならない）
- `serializeXml(nodes)` — `POMNode[]` → XML string (inverse of parseXml; 解決済みの `<Theme>` は保持されない)
- `POMNode` — Input node union type (Text, Ul, Ol, Image, Table, Shape, Chart, Timeline, Matrix, Tree, Flow, ProcessArrow, Pyramid, Line, Arrow, Layer, VStack, HStack, Icon, Svg)

`@hirokisakabe/pom/clientApi` — `parseXml` / `serializeXml` / `POMNode` のみを再エクスポートするクライアント安全なサブパス。`fs` / WASM を含まないため client bundle に含められる。

### Public API (`@hirokisakabe/pom-editor`)

- `PomEditor` — XML / AST editing、preview、diagnostics、共通toolbarを持つReactコンポーネント。preview生成とoptionalなDownload / Save処理はhost callbackへ委譲する。
- `PomAstEditor` — React コンポーネント。`xml` と `onChange` props を受け取り、AST ツリーを表示して DnD でノードを並び替えると更新後の XML を返す。

### Key Internal Types

- `PositionedNode` — Node with absolute position (x, y, w, h)
- Leaf nodes `Text` / `Shape` / `Image` / `Icon` may include `rotate` (degrees clockwise). Rotation is applied in `renderPptx` only; yoga layout uses unrotated bounds.

## Packages


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hirokisakabe/pom](https://github.com/hirokisakabe/pom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
