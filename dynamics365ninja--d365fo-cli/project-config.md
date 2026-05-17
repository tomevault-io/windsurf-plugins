---
trigger: always_on
description: Mirrors the X++ rule canon from `d365fo-mcp-server`'s system prompt
---

# D365 Finance & Operations X++ Development — `d365fo` CLI

<!--
  Mirrors the X++ rule canon from `d365fo-mcp-server`'s system prompt
  (src/prompts/systemInstructions.ts) and `.github/copilot-instructions.md`.
  When that upstream repo updates the X++ rules, sync them here.
  References in the form `[learn:<page>]` link to Microsoft Learn pages
  (see "Authoritative X++ syntax source" at the bottom).
-->

This workspace is the **`d365fo` CLI** — a metadata-aware command-line tool for D365 Finance & Operations. It exposes the same knowledge as `d365fo-mcp-server` but as deterministic shell commands instead of MCP tool calls. Use it whenever you need to reason about a D365FO codebase without burning conversation tokens on long XML or AOT dumps.

> **Audience.** This file gives an AI assistant (GitHub Copilot, Claude Code, Codex CLI, …) the rules for using the `d365fo` CLI to author X++ correctly. Human-facing docs live in the [d365fo-cli repository](https://github.com/dynamics365ninja/d365fo-cli).

---

## 🚨 Core principle — never guess D365FO metadata

Your training data is **outdated and incomplete** for D365FO. Every D365FO environment has hundreds of thousands of tables, classes, EDTs, labels — most of them custom or model-specific. **Before generating any X++ code, query the index** through `d365fo` and ground your answer in real names / signatures.

Concretely: the `d365fo` CLI ships with a SQLite mirror of the AOT (and on Windows D365FO VMs, a live bridge to `IMetadataProvider` + `DYNAMICSXREFDB`). It returns:

- ✅ Real-time metadata from the user's environment (when bridge is running).
- ✅ Pre-indexed SQLite mirror as fallback (Linux/Mac, Azure pipelines, write-only mode).
- ✅ JSON envelopes (`ToolResult<T>`) — fast to parse, cheap on context.

## 🔌 Read-path policy (how `d365fo` resolves info)

`get` / `find` / `search` commands consult sources in this order:

1. **C# bridge** — live `IMetadataProvider` from a running D365FO instance (Windows VM only). Authoritative when available.
2. **SQLite symbol index** — pre-built mirror under `~/.d365fo/index.sqlite` (or `--db`). Used when the bridge is offline.
3. **Filesystem parse** — last resort for objects created in the current session and not yet indexed.

You never pick the source manually. If a result includes `warnings: ["served-from-index"]` the bridge was unavailable and the tool already fell back. If a result is `ok: false` with code `*_NOT_FOUND`, **stop and ask** — do not invent a name.

## 🛡️ Write-path safety

Every `d365fo generate …` command writes XML atomically (`.tmp` + move; `.bak` retained when `--overwrite`). Two modes:

| Mode | When | What happens |
|---|---|---|
| `--out <PATH>` | Standalone scaffolds | Writes to the path you give. Path must be inside the workspace. |
| `--install-to <Model>` | Bridge-installed | Asks the bridge for the model's on-disk location and composes `<modelFolder>/Ax<Type>/<Name>.xml`. Requires `D365FO_BRIDGE_ENABLED=1`. |

Generated files live under `PackagesLocalDirectory/<Model>/<Model>/Ax<Type>/<Name>.xml` — the canonical D365FO layout that Visual Studio and the build tools expect.

---

## 🏁 Mandatory first steps

1. **Verify the index is healthy.**
   ```sh
   d365fo doctor --output json
   d365fo index status --output json
   ```
   - `ok: false` with `code: NO_INDEX` → run `d365fo index extract` first.
   - `warnings: ["stale-index"]` → run `d365fo index refresh` (incremental).

2. **Ground the active model.** When generating files into a workspace, pass either `--install-to <Model>` or `--out <PATH>` explicitly. Never guess the model from search results — ask the user if unclear.

3. **Stale index = wrong answers.** When the user has just edited an XML file, `d365fo get` may return pre-edit data until you re-extract. If results contradict what the user just wrote, re-run `d365fo index refresh --model <Model>`.

---

## ✏️ Editing D365FO files

The CLI's surface is *generation-first*: it scaffolds new XML; it does **not** yet provide an in-place AOT-aware mutator equivalent to MCP's `modify_d365fo_file`. For incremental edits to existing AOT XML you have two options:

1. **Hand-edit XML** with the regular editor tools (`replace_string_in_file`, etc.). Always re-run `d365fo get <kind> <name>` afterwards to confirm the index sees your change once `index refresh` is run.
2. **Drop in a fresh scaffold** with `d365fo generate <kind> --install-to <Model> --overwrite` when you really want to replace the whole file.

When the user just wants to *understand* code, never edit:

| Need | Command |
|---|---|
| Read a class's methods | `d365fo get class <Name> --output json` |
| Read a table's fields / indexes / relations | `d365fo get table <Name> --output json` |
| Read X++ method body | `d365fo read class <Name> --method <M>` |
| Find existing CoC wrappers on a class/method | `d365fo find coc <Class>::<method> --output json` |
| Find event handlers for a target | `d365fo find handlers <Target> --output json` |
| Find which Microsoft pattern peers already use | `d365fo find form-patterns --table <T> --output json` |
| Find forms similar to a known one | `d365fo find form-patterns --similar-to <Form> --output json` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dynamics365ninja/d365fo-cli](https://github.com/dynamics365ninja/d365fo-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
