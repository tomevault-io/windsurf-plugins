---
trigger: always_on
description: The bridge can drive three coding agents: Claude Code, OpenAI Codex and xAI's Grok Build. This page is the reference for each: what to install, how the bridge starts it, what its permission settings mean, and what it cannot do. The code is `bridge/agents.js`, one entry per agent; `bridge.js` only knows the interface (build the arguments, hand over the prompt, read the stream).
---

# Agents

The bridge can drive three coding agents: Claude Code, OpenAI Codex and xAI's Grok Build. This page is the reference for each: what to install, how the bridge starts it, what its permission settings mean, and what it cannot do. The code is `bridge/agents.js`, one entry per agent; `bridge.js` only knows the interface (build the arguments, hand over the prompt, read the stream).

## Choosing one

- `agent` in `bridge/config.json` is the default for every chat (`claude` unless you change it). The bridge refuses to start on a name it doesn't know.
- A chat can pick its own with `/wow-ai agent codex`, or right-click the chat in the left panel and choose **Agent...**. `/wow-ai agent default` goes back to the bridge's. A new chat inherits the agent of the chat you were in, like the folder.
- The choice travels with each message as an `agent=` flag in the strip record, so the bridge needs no restart, and the reply comes back tagged with the agent that wrote it: the bubble label, the `[Codex · chat]` prefix in the game chat and the `To Grok [chat]:` header of `/r` all follow it.
- A session belongs to the agent (and the folder) that made it. A chat that changes agent starts a fresh session with the new one; the transcript in the window stays.
- The bridge's banner lists every agent with the executable it found, or what to install. A chat whose agent is missing gets a reply saying so instead of a hang.

All of them run headless on the bridge PC, so log in once by hand in a terminal there (`claude`, `codex`, `grok login`, `agy`, `hermes setup`); the bridge reuses the cached login.

## The shared vocabulary

Every agent's block in `config.json` has the same keys, and the bridge maps them to that agent's own switches:

| Key | Meaning |
|---|---|
| `permissionMode` | `acceptEdits` (default): edits inside the project are fine, commands need a rule. `default`: nothing is pre-approved beyond reading. `bypassPermissions`: everything is allowed. |
| `allowedTools` | Rules in Claude Code's syntax: `WebSearch` allows a tool, `Bash(git:*)` any command starting with `git`, `Bash(npm test)` that exact command. The **Allow & retry** button appends to this list. Ignored by Codex. |
| `deniedTools` | Rules the agent may never use, same syntax. Claude gets them as `--disallowedTools`, Grok as `--deny` (which wins over everything, `bypassPermissions` included). Ignored by Codex. |
| `model` | Passed through to the CLI when non-empty. |
| `path` | The executable, when the bridge can't find it on its own. A `.js` path is run with the bridge's own Node; a Windows `.cmd` npm shim is unwrapped. |
| `extraArgs` | Anything else to put on the command line, verbatim. |

How each one finds its executable on Windows: a configured `path`; else the installer's folder (`%UserProfile%\.local\bin\claude.exe`, `%UserProfile%\.grok\bin\grok.exe`); else `<name>.exe` on the `PATH`; else npm's `<name>.cmd` launcher, which the bridge unwraps (Node can't spawn `.cmd` files, and going through `cmd.exe` would mangle a system prompt) into the script it runs or the native binary next to it.

Codex also checks `CODEX_BIN` before searching `PATH`, so a newer launcher can override an older `codex.exe` earlier on `PATH`.

## Claude Code

- **Install:** [claude.com/claude-code](https://claude.com/claude-code). Run `claude` once and log in. `claude --version` must work in a terminal.
- **Command line:** `claude -p --output-format stream-json --verbose --permission-mode <mode> --allowedTools <rules…> [--model <m>] [--resume <session>] [--append-system-prompt <context>]`, prompt on stdin. The `CLAUDECODE` variable is removed from the environment so a bridge started from inside a Claude Code session can still launch it.
- **Permissions:** `permissionMode` and `allowedTools` are Claude's own concepts, passed as they are. Denied tools come back in the result as `permission_denials`; the bridge turns them into rules (`Bash(cargo:*)`, `WebSearch`) for the **Allow & retry** button.
- **Session:** the `session_id` on the stream; `--resume` on later runs. Claude keeps sessions per folder.
- **Progress:** one line per `tool_use` block (`edit player.gd`, `$ npm test`, `search: …`) and each text block as a snippet.
- **Context:** the game context and primer go in as a system prompt, on every run.

## Codex

- **Install:** `npm install -g @openai/codex`, then `codex` once to log in (ChatGPT account or API key). On Windows npm installs a `codex.cmd` launcher; the bridge finds the native `codex.exe` inside the package and runs that directly.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chelinho139/wow-ai](https://github.com/chelinho139/wow-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
