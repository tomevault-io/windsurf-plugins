---
trigger: always_on
description: CodeRabbit reads this file as review criteria (its `code_guidelines` default, left on in `.coderabbit.yaml`), so a rule written here is enforced on pull requests, and a stale one misdirects them.
---

# CLAUDE.md

CodeRabbit reads this file as review criteria (its `code_guidelines` default, left on in `.coderabbit.yaml`), so a rule written here is enforced on pull requests, and a stale one misdirects them.

## Commands

```bash
uv sync --frozen --all-extras --dev   # setup (CI uses exactly this)
uv run pytest -v -n auto              # full suite (CI uses -n auto everywhere)
uv run pytest tests/test_cst.py -v    # one file (no -n: worker startup dwarfs the run)
uv run pytest -k test_name            # one test
uv run ruff check .                   # lint
uv run ruff format --check .          # format check (CI enforces)
uv run pyright                        # type check (basic mode)
uv run python mcp_server_kicad/_cst.py  # CST self-check (demo() asserts)
```

Tests that shell out to `kicad-cli` (ERC, DRC, exports) auto-skip when it is not installed; so does the autouse fixture in `tests/conftest.py` that validates every generated `.kicad_sch` and `.kicad_pcb` is parseable by `kicad-cli`. Confirm `kicad-cli` resolves locally (`KICAD_CLI_PATH`, PATH, the macOS app bundle, or the versioned Windows install roots) or a green run proves less than it looks. Even then, a KiCad 9 `kicad-cli` skips KiCad 10 boards (`_cli_can_load_board`), so those are validated only on the macOS and Windows runners. Tests that intentionally write invalid files use `@pytest.mark.no_kicad_validation`.

That validation fixture is what makes the suite slow: it spawns a `kicad-cli` process per generated schematic, ~380 ms each. It is memoised on the file's UUID-normalised digest, so each distinct schematic body is validated once per process rather than once per test (376 spawns -> 186, measured 2026-08-10); `test_validation_memo.py` guards the key against collapsing anything but a UUID. With `-n auto` on top, a full local run went 213 s -> 40 s on 16 cores.

Run a server by hand: `uv run mcp-server-kicad` (stdio), or under the MCP Inspector with `npx @modelcontextprotocol/inspector uv run mcp-server-kicad`. README.md's Inspector command and `.mcp.json` both launch `uvx --from mcp-server-kicad`, which is a PyPI release, not this checkout, so the `kicad` MCP tools in a Claude Code session here do not see local changes.

## Architecture

Five `MCPServer` servers, one module each: `schematic.py`, `pcb.py`, `project.py`, `symbol.py`, `footprint.py`. Each defines its own `mcp` instance and `main()` console script; `server.py` merges all tools into the unified `kicad` server through `_copy_tools`, which reaches into the private `_tool_manager._tools` (no public copy API; `test_tool_annotations.py` depends on the same internals). Every tool passes one of the annotation presets from `_shared.py` (`_READ_ONLY`, `_ADDITIVE`, `_DESTRUCTIVE`, `_EXPORT`) and returns a Pydantic model from `models.py` when it has structured output. The one exception to the presets is `autoroute_pcb`, whose own `_AUTOROUTE` sets `open_world_hint` because it downloads Freerouting. `test_tool_annotations.py` classifies every tool by name, so a new tool needs a row there.

### One parse/write stack: the CST substrate (`_cst.py`)

Stdlib-only, byte-preserving concrete syntax tree for s-expressions. Bytes in, bytes out; `serialize(parse(b)) == b` by construction; malformed input raises instead of being repaired. As of slice 18 every tool's own reads and writes run on it: all `.kicad_sch` reads and writes, all board reads and writes including the `autoroute_pcb` internals, and the symbol/footprint library tools. Two writes are still performed by a subprocess and are outside it (`sym upgrade` and `fp upgrade`, permanently); see the invariant note below. One more design file comes out of a subprocess, as new output rather than an edit: `autoroute_pcb`'s `<stem>_routed.kicad_pcb`, which pcbnew's `SaveBoard` writes in pcbnew's own format while the input board stays untouched. Slice 18 deleted the version guard, so the substrate refuses no file on its format version and every tool's own reads and writes work on both KiCad 9 and KiCad 10 files. What a tool hands to an installed KiCad is another matter: `kicad-cli` (ERC, DRC, exports, jobsets, the library upgrades, `update_pcb_from_schematic`'s netlist export) and pcbnew (`fill_zones`, `autoroute_pcb`) can only work on files that install can load, and neither a KiCad 9 `kicad-cli` nor a KiCad 9 pcbnew can load a KiCad 10 board (both measured; see `_cli_can_load_board` in `tests/conftest.py` and `_require_pcbnew_era` in `pcb.py`). kiutils is a **test-only** dependency (the `dev` extra): it builds fixtures and reads written files back as an independent oracle, and nothing under `mcp_server_kicad` imports it (`test_runtime_imports_no_kiutils` scans for that).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ProductOfAmerica/mcp-server-kicad](https://github.com/ProductOfAmerica/mcp-server-kicad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
