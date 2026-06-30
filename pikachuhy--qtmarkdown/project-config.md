---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
# Configure (adjust CMAKE_PREFIX_PATH to your Qt installation)
cmake -B build -DCMAKE_PREFIX_PATH=<path-to-qt> -DQT=ON

# Build
cmake --build build

# Build with tests enabled
cmake -B build -DCMAKE_PREFIX_PATH=<path-to-qt> -DQT=ON -DBUILD_TEST=ON
cmake --build build

# Run all tests
ctest --test-dir build

# Run a single test
ctest --test-dir build -R test_parser
# Or run the test binary directly:
./build/test/test_parser
# Editor tests need Qt offscreen:
QT_QPA_PLATFORM=offscreen ./build/test/test_editor
```

Tests use the [doctest](https://github.com/doctest/doctest) framework. Some test files define their own `main` via `DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN`; others use `DOCTEST_CONFIG_IMPLEMENT` with a manual `doctest::Context`. Each test binary is self-contained. Test files use `#define private public` / `#define protected public` to access internal members directly.

## Architecture

QtMarkdown has four layers with a strict dependency chain: **Parser → Render → EditorCore → Platform Shells**.

```
QtMarkdownParser  (zero Qt dependency, pure C++26)
    ↑
QtMarkdownRender  (depends on Qt::Gui, MicroTeX)
    ↑
QtMarkdownEditorCore  (platform-agnostic editing logic)
    ↑
QtMarkdownPlatform → QtWidgetMarkdownEditor / QtQuickMarkdownEditor
```

### 1. Parser (`src/parser/` → `QtMarkdownParser`)

Parses Markdown text into an AST using a two-pass approach:

- **Tokenization**: `parseLine()` in `Parser.cpp` splits a line into `Token` objects (type + offset/length into the original text). Special characters `#`, `*`, `~`, `` ` ``, `$`, `[]`, `()`, `!`, `>` become typed tokens; everything else is `text`.
- **Block parsing**: Block parsers are `std::function<ParseResult(const LineList&, int)>` registered in a static vector in `ParserPrivate::parse()`, tried in order until one succeeds. `parseParagraph` is last as the fallback. Each block parser internally registers its own static vector of `LineParserFn` for inline parsing. `ParseResult` = `{ bool success; int offset; unique_ptr<Node> node }` — if `success` is true, the parser consumed `offset` lines and produced an AST `node`; if false, the next parser is tried.
- **Inline parsing**: `LineParserFn` = `std::function<ParseResult(const TokenList&, int)>`. Inline parsers live in `src/parser/parsers/` (`ImageParser`, `LinkParser`, `InlineCodeParser`, `InlineLatexParser`, `SemanticTextParser`).
- **AST nodes**: `Node` base class with `NodeType` enum (in `Node.h`). `Container` nodes hold children via `NodePtrList` = `std::vector<std::unique_ptr<Node>>`. Leaf nodes: `Text`, `Image`, `Link`, `InlineCode`, etc., in `src/parser/nodes/`. The classic **Visitor pattern** is used: `NodeVisitor` (in `Visitor.h`) declares a `visit()` overload for every concrete node type; each node implements `accept(NodeVisitor* v) { v->visit(this); }`.
- **List nodes**: `ListNode` and `ListItemNode` (in `nodes/ListNode.h`) extend `Container`; concrete list classes inline `accept()` to avoid diamond inheritance.
- **PieceTable** (`PieceTable.h`): `PieceTableItem` struct backs `Text` nodes for incremental edits. Each `Text` holds a list of `PieceTableItem` entries referencing spans (offset+length) in either `Document::m_originalBuffer` or `Document::m_addBuffer`. When text is inserted, new content goes into the add buffer and a new `PieceTableItem` points to it — the original text is never modified.
- **Node virtual methods**: `Node` declares three virtual methods used by the editor for cursor-to-markdown-position mapping:
  - `contentLength(doc)` — rendered content length (excludes markup delimiters)
  - `serializedLength(doc)` — markdown length (includes `**`, `*`, `#` prefixes, etc.)
  - `calcMarkdownOffset(doc, contentPos, mdPos)` — maps a content position to a markdown position
  - `clone()` — deep copy, used for snapshot-based undo
  Each of the 22+ concrete node types overrides these as needed. Types without editable content (Hr, Lf, Table) use the default `return 0` / `return false` implementations.
- **ParseContext** (`ParseContext.h`): `thread_local ParseContext` struct tells the `Text(offset, length)` constructor which PieceTable buffer to reference (`original` or `add`). Set by `ParserPrivate::parse()` via RAII `ParseContextGuard`. This lets the editor re-parse add-buffer content without modifying every inline parser.

Key types from `mddef.h`: `String` = `md::String` (custom UTF-8 wrapper around `std::string`, defined in `MdString.h`), `SizeType` = `int64_t`, `sptr<T>` = `std::shared_ptr<T>`, `Char` = `char`. Block/inline parser function signatures are in `ParserDetail.h`.

Each module layers additional type aliases:
- `src/render/mddef.h`: `InstructionPtr`, `InstructionPtrList`, plus re-exports from `editor/core/Types.h` (`Color`, `Point`, `Size`, `Rect`, `Font`)
- `src/editor/core/Types.h`: `Point`, `Size`, `Rect`, `Color`, `Margins`, `FontDescription`, `ImageData`
- `src/editor/core/Event.h`: `KeyEvent`, `MouseEvent`
- `src/editor/core/AbstractPainter.h`: abstract `Painter` interface
- `src/editor/core/Timer.h`: `Timer` abstraction

### 2. Render (`src/render/` → `QtMarkdownRender`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PikachuHy/QtMarkdown](https://github.com/PikachuHy/QtMarkdown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
