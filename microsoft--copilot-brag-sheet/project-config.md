---
trigger: always_on
description: > **README for AI coding agents.** If you're a human looking for setup or
---

# AGENTS.md

> **README for AI coding agents.** If you're a human looking for setup or
> usage instructions, read [`README.md`](README.md) instead. This file is the
> dedicated place where Claude Code, Copilot CLI, Codex, Cursor, Jules, Amp,
> and friends find the context they need to be productive on this repo
> without re-deriving it from grep.
>
> **Format:** [agents.md](https://agents.md/) — used by 60k+ open-source
> projects and stewarded by the Agentic AI Foundation under the Linux
> Foundation. The closest `AGENTS.md` to the file you're editing wins;
> explicit prompts override everything.

---

## 1. Project overview

`copilot-brag-sheet` is a [GitHub Copilot CLI](https://docs.github.com/en/copilot/github-copilot-in-the-cli)
extension that silently records work as it happens — files edited, PRs
created, git actions, manual brag entries — into structured local JSON so a
developer has receipts at performance review time instead of a blank page.
It ships as a [`joinSession()`](https://docs.github.com/en/copilot/customizing-copilot/extending-copilot-with-mcp-and-extensions)
extension installed into `~/.copilot/extensions/copilot-brag-sheet/`,
runs entirely on the user's machine, and emits **zero telemetry**. The
core library (`lib/`) is dependency-free; the cross-engine MCP server
(`mcp-server.mjs`) takes two pinned, audited runtime dependencies — the
official MCP SDK and Zod — to stay protocol-conformant. The user-facing
pitch is:

> Turn vague *what did I do?* into evidence-backed impact statements —
> automatically, every Copilot CLI session.

Public repo: <https://github.com/microsoft/copilot-brag-sheet> ·
npm: [`copilot-brag-sheet`](https://www.npmjs.com/package/copilot-brag-sheet) ·
landing: <https://microsoft.github.io/copilot-brag-sheet/>.

---

## 2. Architecture map

```
Copilot CLI host  ──spawns──▶  extension.mjs (joinSession)
                                    │
                                    ├── hooks: onSessionStart / onUserPromptSubmitted /
                                    │           onPostToolUse / onSessionEnd /
                                    │           session.shutdown
                                    │
                                    ├── tools: save_to_brag_sheet
                                    │          review_brag_sheet
                                    │          generate_work_log
                                    │
                                    └── delegates to lib/* (pure Node, no SDK coupling)
                                            │
                                            ▼
                              JSON records on disk (atomic writes)
                              ~/AppData/Local/copilot-brag-sheet/  (Windows)
                              ~/Library/Application Support/copilot-brag-sheet/  (macOS)
                              ~/.local/share/copilot-brag-sheet/  (Linux, XDG)
                                            │
                                            ▼
                              optional: git commit/push to private repo

Agency host  ──spawns──▶  hooks/post-tool-use.mjs (subprocess per event)
                               │
                               ├── reads JSON payload from stdin
                               ├── classifies via lib/heuristics.mjs
                               └── writes JSON response to stdout
                                   (Phase 1: classification only, no persistence)

MCP host  ──stdio──▶  mcp-server.mjs (@modelcontextprotocol/sdk)
                           │
                           ├── tools: save_to_brag_sheet / review_brag_sheet / generate_work_log
                           └── delegates to lib/operations.mjs
```

**Entry points** (start here):

| File | Role |
|---|---|
| [`extension.mjs`](extension.mjs) | The only file that imports `@github/copilot-sdk/extension`. Wires hooks and tools to `lib/*`. **All Copilot-specific glue lives here and nowhere else.** |
| [`mcp-server.mjs`](mcp-server.mjs) | MCP stdio server exposing all three tools. Uses `@modelcontextprotocol/sdk` + `zod`. Works with any MCP-compatible host. |
| [`hooks/post-tool-use.mjs`](hooks/post-tool-use.mjs) | Agency PostToolUse hook. Classifies tool calls via `lib/heuristics.mjs`, returns classification to host via stdout. **Phase 1: classification only, no persistence.** |
| [`bin/install.mjs`](bin/install.mjs) | `npm i -g copilot-brag-sheet && copilot-brag-sheet` — copies package files into `~/.copilot/extensions/` and runs setup. |
| [`bin/setup.mjs`](bin/setup.mjs) | Interactive wizard: presets, git backup, output path. Non-TTY exits cleanly (CI-safe). |
| [`install.sh`](install.sh) / [`install.ps1`](install.ps1) | Curl-pipe-bash installers. Cross-platform CI-tested on PS 5.1 and pwsh 7. |

**Core library** (`lib/`) — dependency-free, pure Node, Copilot-agnostic
(safe to import from any entry point):

| Module | Responsibility | Key exports |
|---|---|---|
| [`paths.mjs`](lib/paths.mjs) | OS-native data dir + path helpers; respects `WORK_TRACKER_DIR` / `XDG_DATA_HOME` / `LOCALAPPDATA`. | `detectDataDir`, `detectBragSheetPath`, `detectGitConfig`, `ensureDir` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/copilot-brag-sheet](https://github.com/microsoft/copilot-brag-sheet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
