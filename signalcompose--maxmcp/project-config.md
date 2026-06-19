---
trigger: always_on
description: Provides:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MaxMCP is a native C++ external object for Max/MSP that implements an MCP (Model Context Protocol) server. It enables Claude Code to control Max/MSP patches through natural language commands via a unified external with two modes: `@mode agent` (WebSocket server, MCP handler) and `@mode patch` (client registration).

**Architecture**: Claude Code ↔ stdio ↔ Node.js Bridge ↔ WebSocket ↔ maxmcp.mxo ↔ Max/MSP Patches

## セッション開始時の必須アクション

**CRITICAL: C++ コードやスキルドキュメントの作業を開始する前に、利用可能なツールを確認し活用すること。**

### ステップ1: MCP ツール確認

作業内容に応じて以下のツールを活用する:

| ツール | 用途 | 活用場面 |
|--------|------|----------|
| **Serena** | シンボルベースのコードナビゲーション | C++ 実装・リファクタ・バグ修正時。`activate_project` → `get_symbols_overview` → `find_symbol` で構造を把握してから着手 |
| **Context7** | 依存ライブラリの最新ドキュメント参照 | libwebsockets, nlohmann/json, Google Test 等の API を確認する必要がある時 |

### ステップ2: 作業対象の把握

- **C++ コード作業**: 対象ファイルの `get_symbols_overview` でシンボル構造を確認。Grep/Read だけで力技で調べず、Serena のシンボル検索・参照追跡を活用する
- **スキルドキュメント作業**: 対象スキルの SKILL.md と reference/ を確認してから編集
- **ビルド・テスト**: 下記の Build & Deploy セクション参照

### やってはいけないこと

- Serena が利用可能なのに Grep/Read だけでコード構造を調査する
- 依存ライブラリの API を記憶に頼って使用する（Context7 で最新ドキュメントを確認する）
- シンボルの参照箇所を手動検索する（`find_referencing_symbols` を使う）

### なぜこれが重要か

1. **Serena はシンボル単位の正確な操作を提供する**: Grep/Read による文字列検索では、同名の変数やコメント内の一致を拾ってしまう。Serena の `find_symbol` / `find_referencing_symbols` はセマンティックな解析に基づくため、リファクタリング時の影響範囲を正確に特定できる
2. **Context7 は最新の API 仕様を保証する**: libwebsockets や nlohmann/json の API は知識カットオフ以降に変更されている可能性がある。記憶に頼ると非推奨 API の使用やシグネチャの不一致を招く
3. **ツール未活用は手戻りの原因になる**: 力技の調査は見落としが発生しやすく、後工程でのバグや修正漏れにつながる

## Build & Deploy

```bash
# Build (configure + build + install to package/MaxMCP)
./build.sh              # Debug build
./build.sh --test       # Debug build with tests
./build.sh Release      # Release build
./build.sh --clean      # Clean build first

# Deploy to Max 9 Packages (removes old package automatically)
./deploy.sh

# Typical workflow
./build.sh --test && ./deploy.sh
```

### Manual commands (for reference)

```bash
cmake -B build -S . -DCMAKE_BUILD_TYPE=Debug -DBUILD_TESTS=ON
cmake --build build
cd build && ctest --output-on-failure && cd ..
cmake --install build --prefix package/MaxMCP
```

**Test framework**: Google Test 1.17.0
**Test files**: `tests/unit/test_*.cpp`
**Test mode macro**: `MAXMCP_TEST_MODE` (enables compilation without Max SDK)
**Integration test checklist**: [docs/integration-test-checklist.md](docs/integration-test-checklist.md) (manual verification of all 30 MCP tools)

## Code Quality

**C++ formatting**: clang-format (run before commit)
```bash
find src -name "*.cpp" -o -name "*.h" | xargs clang-format -i
```

**Node.js (bridge)**: ESLint + Prettier
```bash
cd package/MaxMCP/support/bridge
npm run lint
npm run format:check
```

## Architecture Overview

### Key Components

- **`src/maxmcp.cpp`**: Unified external object supporting both agent and patch modes via `@mode` attribute
- **`src/mcp_server.cpp`**: MCP protocol handler (JSON-RPC), implements all MCP tools
- **`src/websocket_server.cpp`**: libwebsockets-based WebSocket server for bridge communication
- **`src/tools/`**: MCP tool implementations organized by category (patch, object, connection, state, hierarchy, utility, layout)
- **`src/utils/`**: Shared utilities (UUID generator, console logger, patch registry, patch helpers)
- *Note*: Legacy files `maxmcp_server.cpp`, `udp_server.cpp` from the earlier separate-external architecture have been removed

### MCP Tools (30 total)

Full tool reference with parameters and response formats: [docs/mcp-tools-reference.md](docs/mcp-tools-reference.md)

| Category | Count | Tools |
|----------|-------|-------|
| Patch Management | 3 | `list_active_patches`, `get_patch_info`, `get_frontmost_patch` |
| Object Operations | 12 | `add_max_object`, `remove_max_object`, `get_objects_in_patch`, `set_object_attribute`, `get_object_attribute`, `get_object_value`, `get_object_io_info`, `get_object_hidden`, `set_object_hidden`, `redraw_object`, `replace_object_text`, `assign_varnames` |
| Connection Operations | 4 | `connect_max_objects`, `disconnect_max_objects`, `get_patchlines`, `set_patchline_midpoints` |
| Patch State | 3 | `get_patch_lock_state`, `set_patch_lock_state`, `get_patch_dirty` |
| Hierarchy | 2 | `get_parent_patcher`, `get_subpatchers` |
| Utilities | 2 | `get_console_log`, `get_avoid_rect_position` |
| Layout Validation | 4 | `validate_layout`, `get_io_position`, `suggest_alignment`, `align_objects` |

### Threading Model

All Max API calls must run on the main thread. Use `defer()` for operations triggered from WebSocket callbacks:

```
WebSocket Thread → defer() → Max Main Thread → Execute Max API
```

## Dependencies

- **C++17** (required)
- **Max SDK v8.2.0** (clone to `max-sdk/` directory; newer max-sdk dropped the `max-sdk-base` submodule this project requires)
- **nlohmann/json 3.11.0+** (`brew install nlohmann-json`)
- **libwebsockets** (`brew install libwebsockets`)
- **Google Test** (for tests only, `brew install googletest`)

## Git Workflow

**Branch strategy**: GitHub Flow

- `main` - Production (default branch, protected)
- `feature/XX-description` - Feature branches (XX = issue number)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [signalcompose/MaxMCP](https://github.com/signalcompose/MaxMCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
