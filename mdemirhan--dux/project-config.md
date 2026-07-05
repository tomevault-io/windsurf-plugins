---
trigger: always_on
description: Instructions for AI agents working on this codebase.
---

# Agent Guidelines

Instructions for AI agents working on this codebase.

## Tooling

- Use `uv` for all operations:
  - `uv sync` to install dependencies
  - `uv run <command>` to run commands in the project environment
  - `uv add <package>` to add dependencies
- Use `ruff` for linting and formatting (`ruff check`, `ruff format`)
- Use `basedpyright` for type checking
- Use `pytest` for testing
- After making code changes, run `uv run ruff format`, `uv run ruff check`, and `uv run basedpyright` before considering work complete

## Architecture

```
csrc/
├── walker.c                    # C source: scan_dir_nodes (readdir), scan_dir_bulk_nodes (getattrlistbulk)
├── ac_matcher.c                # C source: Aho-Corasick automaton (trie + fail links + BFS)
└── prefix_trie.c               # C source: Prefix trie for O(basename) startswith matching

docs/
├── architecture.md             # End-to-end scan pipeline documentation
├── aho-corasick.md             # Aho-Corasick algorithm deep dive
└── prefix-trie.md              # PrefixTrie algorithm deep dive

dux/
├── _ac_matcher.so / .pyi   # Compiled C extension: Aho-Corasick multi-pattern matcher
├── _prefix_trie.so / .pyi  # Compiled C extension: PrefixTrie startswith matcher
├── _walker.so / .pyi       # Compiled C extension: scan_dir_nodes (POSIX), scan_dir_bulk_nodes (macOS)
├── cli/app.py              # Entry point, CLI flags, progress display
├── ui/
│   ├── app.py              # TUI application (Textual), all views and keybindings
│   ├── views.py            # View generators: overview_rows, browse_rows, insight_rows, top_nodes_rows
│   └── app.tcss            # Textual CSS styling (Tomorrow Night theme)
├── models/
│   ├── enums.py            # NodeKind (FILE/DIRECTORY), InsightCategory (TEMP/CACHE/BUILD_ARTIFACT)
│   ├── scan.py             # ScanNode, ScanStats, ScanSnapshot, ScanError, ScanResult
│   └── insight.py          # Insight, InsightBundle dataclasses
├── config/
│   ├── schema.py           # AppConfig, PatternRule dataclasses with to_dict/from_dict
│   ├── defaults.py         # 59 built-in pattern rules
│   └── loader.py           # JSON config loading with FileSystem abstraction
├── scan/
│   ├── __init__.py          # Scanner protocol, default_scanner() (GIL-aware selection)
│   ├── _base.py             # ThreadedScannerBase (thread pool + work queue)
│   ├── python_scanner.py    # Pure Python scanner using FileSystem.scandir()
│   └── native_scanner.py    # C extension scanner wrapping scan_dir_nodes / scan_dir_bulk_nodes
└── services/
    ├── fs.py               # FileSystem protocol, OsFileSystem, DEFAULT_FS singleton
    ├── insights.py          # Insight generation: DFS traversal, per-category min-heaps for top-K
    ├── patterns.py          # Compiled matchers: EXACT, CONTAINS+ENDSWITH (AC), STARTSWITH (PrefixTrie), GLOB
    ├── tree.py              # Tree traversal: iter_nodes, top_nodes (heapq.nlargest), finalize_sizes
    ├── formatting.py        # format_bytes, relative_bar, relative_path
    └── summary.py           # Non-interactive CLI summary rendering
```

### Data Flow

1. `cli/app.py` parses CLI args, loads config via `loader.py`, selects scanner via `default_scanner()`
2. The selected scanner (`PythonScanner` or `NativeScanner`) walks the filesystem in parallel, builds `ScanSnapshot` (immutable tree of `ScanNode`)
3. `insights.py` walks the scan tree, matches against compiled patterns, produces `InsightBundle`
4. Either `summary.py` renders CLI output, or `ui/app.py` launches the interactive TUI

### Key Design Decisions

- **`ScanSnapshot` is immutable after scanning.** The scan tree never changes. All TUI views are read-only projections. Row caches are safe to keep across tab switches.
- **`Result[T, E]` for error handling.** Scanner and config loader return `Result` types. CLI/TUI boundary code unwraps them.
- **`FileSystem` protocol for testability.** `PythonScanner` and config loader accept a `fs` parameter (defaults to `DEFAULT_FS` singleton). Tests use `MemoryFileSystem` — no temp files, no disk I/O. Note: `NativeScanner` bypasses `FileSystem` entirely, calling C extensions directly.
- **`DirEntry.stat` is bundled, not separate.** `OsFileSystem.scandir` calls `entry.stat(follow_symlinks=False)` on the `os.DirEntry` object (which uses OS-cached stat data) and bundles the result into each `DirEntry`. The scanner reads `entry.stat` directly — never calls `fs.stat()` per entry in the hot loop.
- **GIL-aware scanner selection.** `default_scanner()` picks the best backend: `NativeScanner(scan_dir_bulk_nodes)` on macOS (uses `getattrlistbulk` — single syscall per directory batch), `NativeScanner(scan_dir_nodes)` when GIL is enabled (C `readdir`, benefits from GIL release during I/O), `PythonScanner` when GIL is disabled (true parallelism makes C overhead negligible).

## Performance-Critical Code

**Any change to `scan/`, `services/fs.py`, or `services/patterns.py` that could affect scanning or pattern matching performance must be flagged to the user before implementation.**

### General Constraints


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mdemirhan/dux](https://github.com/mdemirhan/dux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-05 -->
