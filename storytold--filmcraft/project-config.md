---
trigger: always_on
description: FilmCraft is built to be driven by AI agents, and largely built by them. Part 1 covers using
---

# Agents

FilmCraft is built to be driven by AI agents, and largely built by them. Part 1 covers using
FilmCraft from an agent. Part 2 covers developing FilmCraft as an agent.

# Part 1: driving FilmCraft

## 1. MCP server

`filmcraft-cli mcp` serves MCP on stdio. It has two modes:

| Mode | Command | What it drives |
|---|---|---|
| Headless | `filmcraft-cli mcp --demo` or `--project p.fcproj` (neither = empty project) | an in-process engine session, no window |
| Bridge | `filmcraft-cli mcp --bridge 127.0.0.1:9876` | the running desktop app started with `filmcraft --control 9876` |

**Claude Code.** The repository's `.mcp.json` registers both servers (`filmcraft` = bridge,
`filmcraft-headless` = demo). Both point at `target/release/filmcraft-cli`, so build it first:

```sh
cargo build --release -p filmcraft-cli -p filmcraft
./target/release/filmcraft --control 9876 &      # only for the bridge server
claude                                           # approve the project MCP servers when asked
```

To register the server by hand, or for another MCP client, use the absolute binary path:

```sh
claude mcp add filmcraft-headless -- /abs/path/filmcraft/target/release/filmcraft-cli mcp --demo
```

```json
{ "mcpServers": { "filmcraft": { "command": "/abs/path/filmcraft-cli", "args": ["mcp", "--bridge", "127.0.0.1:9876"] } } }
```

### Tools

| Tool | Modes | Purpose |
|---|---|---|
| `command_list` | both | every command: id, label, menu, shortcut, params, enabled now (`filter`, `enabled_only`) |
| `command_run` | both | run a command `{id, params}`; edits are undoable |
| `command_batch` | both | run several commands in order `{steps: [{id, params}], stop_on_error}` → `{completed, failed, results}` |
| `doc_inspect` | both | the project tree and the active sequence in one call (`project_inspect` + `sequence_inspect`) |
| `render_preview` | both | same as `render_frame` |
| `project_inspect` | both | bins and items with ids, types, durations; active sequence |
| `sequence_inspect` | both | the active sequence: tracks, clips (`start` / `end` and `sourceIn` / `sourceOut` in ticks, frames, `speed`, `reverse`: the clip plays its `sourceIn` to `sourceOut` stretch backward, `gainDb`), effects, transitions, markers (name and `comment`), playhead, selection |
| `media_import` | both | import files by absolute path (`text`, one path per line) |
| `render_frame` | both | PNG of the program frame at `seconds` (headless renders; bridge screenshots the Program monitor) |
| `ui_inspect` | bridge | UI state: tool, workspace, panels, zoom, playback, fps |
| `ui_elements` | bridge | on-screen interactive elements with id, label and rect (`prefix` filter) |
| `ui_click` | bridge | click by `id` or `x`,`y`; `button`, `count`, `modifiers` |
| `ui_drag` | bridge | press–move–release between points or element ids |
| `ui_key` | bridge | key or shortcut: `Space`, `Cmd+K`, `Shift+Delete` |
| `ui_type` | bridge | type text into the focused field |
| `ui_screenshot` | bridge | PNG of the window or one `panel` |
| `ui_control` | bridge | call any control-channel method directly (`ui.set`, `ui.scroll`, `ui.timeline.locate`, …) |

Typical loop: `project_inspect` / `sequence_inspect` → get ids → `command_run` → `render_frame` or
`ui_screenshot` → look at the result → `edit.undo` if it's wrong.

### Conventions

The server follows a few conventions that make it predictable for agents. They are not
FilmCraft-specific: the sibling craft apps' MCP servers can follow the same ones, so an agent that has
driven one app can drive the others.

- **Core tools.** `command_list`, `command_run`, `command_batch`, `doc_inspect` and `render_preview`
  work in both modes. The FilmCraft tools above stay as they are.
- **Titles and annotations.** Every tool has a `title` and the four MCP hints (`readOnlyHint`,
  `destructiveHint`, `idempotentHint`, `openWorldHint`), so a client can let the read-only tools
  (lists, inspects, renders, screenshots) run without asking and ask before edits. When you add a
  tool, set them in its `#[tool(...)]` attribute; a test checks every listed tool.
- **Strict arguments.** An unknown tool argument is a JSON-RPC `-32602` error that names it and the
  accepted ones (`unknown argument "filtr" for command_list; expected: enabled_only, filter`), not a
  silently ignored key. `command_run.params` goes to the command unchanged.
- **Errors.** A failing command is an `isError` result with the engine's message. A line that is not
  JSON gets a `-32700` parse error (id `null`) and the server keeps serving. A command that panics is
  reported as `internal error: …`, and the session stays usable.
- **Resources.** `filmcraft://document` (the project tree and active sequence, as `doc_inspect`) and
  `filmcraft://commands` (the command catalog, as `command_list`), both `application/json`. For MCP
  2026-07-28 clients such as current Claude Code, list and read results carry the `ttlMs` and
  `cacheScope` hints that revision requires.

### Long exports: progress and cancellation

`command_run {"id": "file.exportMedia", "params": {…, "wait": true}}` in headless mode blocks until
the file is written. It uses the MCP progress and cancellation utilities, so a client can show
progress and stop it:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [storytold/filmcraft](https://github.com/storytold/filmcraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
