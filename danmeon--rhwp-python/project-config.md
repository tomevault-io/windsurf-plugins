---
trigger: always_on
description: Project-specific instructions. Inherits all rules from `~/.claude/CLAUDE.md` (global).
---

# AGENTS.md — rhwp-python

Project-specific instructions. Inherits all rules from `~/.claude/CLAUDE.md` (global).

## Project context

- **What it is**: PyO3 Python bindings for the [edwardkim/rhwp](https://github.com/edwardkim/rhwp) Rust HWP/HWPX parser & renderer
- **Names**: PyPI `rhwp-python` / `import rhwp` / extension `rhwp._rhwp`
- **Core delivery**: Rust core consumed via git submodule at `external/rhwp`, pinned to a specific upstream commit (tracked in `CHANGELOG.md` + `.gitmodules`)
- **License**: MIT — dual copyright (Edward Kim for rhwp core, DanMeon for bindings). Both LICENSE files are bundled in the wheel (`license-files = ["LICENSE", "external/rhwp/LICENSE"]`)
- **Status**: unofficial community package. The `rhwp` name on PyPI is intentionally left for the upstream maintainer

## Project-specific rules

### Rust + Python hybrid build
- After any Rust change (`src/*.rs`): `uv run maturin develop --release` before `pytest` (without it, tests run against the stale binary), and `cargo clippy --all-targets -- -D warnings` for lint
- `external/rhwp/` is upstream-owned. Never edit it locally — file an issue / PR against [edwardkim/rhwp](https://github.com/edwardkim/rhwp) instead
- PyO3 `#[pyclass(unsendable)]`: `_Document` is bound to its creation thread (upstream `DocumentCore` holds `RefCell` fields — `!Sync`). Same-thread worker pattern (`parse + consume + return primitives` inside one thread) works; `asyncio.to_thread(rhwp.parse, path)` does NOT — the Future resolves on the main thread and first attribute access panics with `_rhwp::document::PyDocument is unsendable, but sent to another thread`
- GIL release via `py.detach` — apply selectively, not blanket:
  - **Release** for ≥1 ms CPU/IO-bound work that touches only Rust-side data (parse, render, decode, compress, file read). Current sites: `_Document::from_bytes` / `render_pdf()` / `export_pdf()`. When adding new methods of this shape, follow the same pattern
  - **Don't release** for trivial getters, short attribute access, or hot paths that frequently call back into Python — the `detach`/`attach` round-trip cost exceeds the gain, and may slow things down
  - **When unsure**, measure with the `benches/bench_gil.py` pattern (with vs without `py.detach` wall-clock comparison) before committing
- `abi3-py310` feature: **one wheel covers 3.10–3.13+**. Don't bind to Python version-specific C API

### Async direction
- Python-surface APIs for I/O and integrations are **async-first**: when adding LangChain (or future RAG framework) loaders, implement `aload` / `alazy_load` / async counterparts alongside sync versions
- **Forbidden pattern**: `asyncio.to_thread(rhwp.parse, path)` — `_Document` is unsendable (see Rust+Python hybrid build note above), the returned Document panics on main-thread access. `async fn` in `#[pymethods]` is also incompatible (PyO3 requires `Send + 'static` futures)
- **Supported async pattern**: `aparse(path)` uses stdlib `asyncio.to_thread` to offload the file read to a thread pool, then calls `Document.from_bytes(data)` on the event-loop thread. Document never crosses a thread boundary. No external dependency — Python `asyncio` lacks native async file I/O so all async file libs (aiofiles etc.) wrap thread pools anyway; stdlib achieves the same effect with zero install footprint
- **Document instance-level async methods (`doc.ato_ir()` etc.) are NOT provided** — they would require thread offload which unsendable forbids. For async code, `await rhwp.aparse(path)` once, then call sync methods on the Document directly (these are fast, in-memory, GIL-holding operations)
- If upstream rhwp ever replaces its `RefCell` caches with thread-safe synchronization, revisit this — `unsendable` could then be dropped, enabling true `async fn pymethods`

### Tests
- Real HWP fixtures live in the submodule: `external/rhwp/samples/aift.hwp` (HWP5), `table-vpos-01.hwpx` (HWPX). `tests/conftest.py` + `benches/bench_gil.py` reference this path
- When changing one path, change both
- Markers: `slow` (PDF render), `langchain` (extras required). Default run: `pytest -m "not slow"`
- Extras-gated test files use module-level `pytest.importorskip` so the whole file counts as **1 skip** when the extra is missing. Current gated files: `test_langchain_loader.py` + `test_langchain_loader_ir.py` (langchain-core), `test_ir_schema_export.py` (jsonschema), `test_cli.py` (typer), `test_mcp_server.py` (fastmcp), `test_render_png.py` (Pillow) → CI's `test-without-extras` job validates **exactly 6 skipped** (see `.github/workflows/ci.yml`). When adding a new extras-gated file, bump the count in both AGENTS.md and ci.yml
- `tests/type_check_errors.py` holds **exactly 4 intentional pyright errors** — CI validates that too. When editing, preserve count; don't fix them

### Git workflow
- Single-branch trunk model: feature branches off `main` → PR to `main`. No `develop` / `staging`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DanMeon/rhwp-python](https://github.com/DanMeon/rhwp-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
