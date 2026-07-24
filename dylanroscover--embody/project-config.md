---
trigger: always_on
description: **Embody** is a TouchDesigner extension that automates externalization of COMP and DAT operators to version-control-friendly files (.tox, .py, .json, .xml, etc.). It solves the problem of TouchDesigner's binary .toe files being impossible to diff/merge in git.
---

# Embody + Envoy

## Project Overview

**Embody** is a TouchDesigner extension that automates externalization of COMP and DAT operators to version-control-friendly files (.tox, .py, .json, .xml, etc.). It solves the problem of TouchDesigner's binary .toe files being impossible to diff/merge in git.

**Envoy** is an MCP (Model Context Protocol) server embedded inside Embody that lets Claude Code create, modify, connect, and query TouchDesigner operators programmatically -- plus manage Embody externalizations.

## Critical Rules

1. **Prefer `.tdn` files for reading TDN-externalized COMPs** -- `.tdn` files are YAML on disk with complete network structure (operators, parameters, connections, positions, flags, DAT content, annotations). Reading them directly is faster than MCP round-trips. Check `externalizations.tsv` (strategy column) or call `get_externalizations` to identify TDN-strategy COMPs. To edit: modify the `.tdn` file on disk, then **always** call `import_network` via MCP with the COMP path, the parsed network, and `clear_first=True` to reload it in TD. **Never leave a `.tdn` edit unreloaded** -- the user must see updates immediately in TD. Use MCP when you need live runtime state (evaluated expressions, cook errors) or for non-TDN operators.
2. **Use Envoy MCP tools for live TD state and non-TDN operators** -- NEVER say "I can't edit that because it's in a .tox" or "these are binary files I can't access." For operators not externalized as TDN, use MCP tools to inspect and modify them. The filesystem holds externalized files (`.py`, `.tox`, `.tdn`, `.json`, `.xml`, etc.); MCP is for interacting with live operator state inside TD.
3. **NEVER create operators under `/local`** -- `/local` is volatile storage, not saved with the `.toe` file. Place new COMPs in the container that holds the `Embody` COMP (`op.Embody.parent().path`, the level the user chose) -- or a network the user has deliberately opened -- never `/local` and never the bare root `/`. See `/create-operator` step 1.
4. **Do NOT assume network paths** -- never guess `/project1`. Use `query_network` on `/` to discover the actual root structure.
5. **Default new COMPs to Embody's container** -- `execute_python` with `result = op.Embody.parent().path` returns the same home every run (the consistency anchor). Build in `ui.panes.current.owner.path` only when the user has deliberately navigated into a content network, and never treat bare `/` as that home. See `/create-operator` step 1.
6. **Always consult the TD wiki** before writing TD Python code OR claiming TD behavior -- confirm API behavior, file formats, and application features against official Derivative documentation even if you're confident. Never assume a TD feature, file type, or convention exists without a verified source.
7. **Binary files** (`.toe`, `.tox`) -- use MCP tools to inspect contents, not the filesystem.
8. **Always check for errors after creating operators** -- `get_op_errors` with `recurse=true` immediately after creating and connecting operators.
9. **Favor annotations over OP comments** -- use `create_annotation` for documenting operators and groups.
10. **Always analyze log files after MCP operations** -- read `dev/logs/` for the complete picture. Ring buffer only holds 200 entries.
11. **Always update unit tests when modifying project code** -- check whether existing tests assert against changed behavior.
12. **Batch repetitive MCP operations** -- never make 3+ individual calls to the same tool. Use `batch_operations` to combine `set_op_position`, `connect_ops`, `set_parameter`, `set_op_flags`, etc. into a single request. For complex logic (conditionals, loops, computed values), use `execute_python` instead. Each MCP round-trip costs tokens and latency -- minimize them.
13. **Prefer the operator-creating MCP tools** (`create_op`, `copy_op`, `create_extension`) over raw `execute_python` -- they auto-position, lint layout, and (when the Envoy `Autoexternalize` preference is `DATs`/`COMPs`/`both`) auto-externalize new COMPs (TDN) and DATs (source) at their boundary -- additively, never inside an already-externalized ancestor. A `copy_op` gets a **fresh** externalization at its own path (inherited source tags/file-refs are cleared, so the copy never shares or overwrites the source's files); `create_extension` externalizes the host COMP it creates (its code DAT is captured inside). Batch via `batch_operations`. Reach for `execute_python`/`comp.create()`/`.copy()` only when you genuinely need computed/looped creation or connection-preserving `copyOPs`; those bypass auto-externalization (Envoy rides an `AUTO-EXTERNALIZE BYPASS` warning back on the response) and require manual layout + tagging.

## Approach Guidelines

- Before editing a file, verify it is the ACTUAL file responsible. Grep and trace the render path before making changes.
- Avoid over-engineering. Prefer minimal, targeted changes.
- When debugging, state your hypothesis, verify with evidence, then fix.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dylanroscover/Embody](https://github.com/dylanroscover/Embody) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
