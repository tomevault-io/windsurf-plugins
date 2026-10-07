---
trigger: always_on
description: DeskVNC exposes an optional control plane so an existing AI agent can observe and
---

# Driving DeskVNC from an AI agent

DeskVNC exposes an optional control plane so an existing AI agent can observe and
act on the machines you connect to, over VNC, RDP or SSH, with nothing installed
on the remote target. This document is the practical entry point. The design
rationale lives in the source comments and the internal specification cited there
as `PRDAgentPlug/NN`.

## The shape of it

The plane follows one loop:

```
observe()  ->  Observation      a screenshot plus geometry and session state
decide()   ->  Action           the agent's business, not ours
act()      ->  ActionResult     one settlement per action, never fire and forget
```

Three properties separate it from the version a person writes in an afternoon:

- **The observation is fenced, twice.** It carries a geometry generation, and an
  action computed against a stale geometry is refused rather than landing in the
  wrong place after a resize. It also carries a content generation, and text
  typed into a screen the agent has not read since something large repainted is
  refused rather than going into whatever window appeared over the one it was
  looking at.
- **The action is settled, not fired.** Every intent gets an id and exactly one
  result, so an agent never waits forever on something a driver could not serve.
- **The loop can lose the machine mid step.** A person can take the wheel between
  observe and act. Control is leased; on any lease change the plane releases all
  held keys, so a half-finished drag cannot strand the desktop.

## Transports

`dvv` is one server behind two transports:

- **stdio**, for agents that spawn a subprocess. This is the default. `dvv`
  reaches the application over a unix socket on macOS and Linux, and over a
  named pipe on Windows (`\\.\pipe\deskvncviewer-agent-<user>`) whose ACL
  admits only the user who started DeskVNCViewer.
- **HTTP**, for agents that cannot. It is off by default, binds to loopback,
  always requires a bearer token, checks `Origin`, and refuses to start rather
  than start without a token.

Both frame the same dispatch table, so a tool behaves identically regardless of
how the agent reached it.

## Getting connected

There is nothing to do. Install DeskVNCViewer, open it once, and save a
machine with its password. Each time the app starts it finds the agents
installed on the computer (OpenCode, Pi, Claude Code and Codex) and connects
each one to it. Then ask your agent to do something on one of your machines by
name. It opens the machine itself, recovers it if the screen stalls, and starts
DeskVNCViewer if it is closed.

An agent installed after the app last started is picked up on the next launch,
or straight away with **Connect now** in the AI Agents panel. An agent that was
already open when it was connected needs a restart to see the new tools. After
an update, the next launch points every agent at the new `dvv`.

On macOS and Linux an app opened from the Dock or a desktop menu has none of
the PATH your terminal has, so setup reads your login shell's PATH and also
looks where nvm, fnm, Volta, asdf, mise, Homebrew and npm put commands. An
agent installed any of those ways is found. The Linux AppImage runs from a
folder that disappears when it quits, so its `dvv` is copied to
`~/.local/share/DeskVNCViewer/bin/dvv` and agents are pointed there.

All of this is checked on every change by the "Agents end to end" workflow,
on macOS, the Linux `.deb` and the AppImage. It installs OpenCode and Pi
through nvm, opens the app once, and has both drive a machine with the app
closed.

The plane is on unless somebody switches it off in the AI Agents panel, and
off is remembered. It is reachable only by the user who started the app.

The same wiring from a terminal is `dvv setup`, or `dvv setup opencode` (or
`pi`, `codex`, `claude`) for one agent. `dvv` sits beside the app: in
`/Applications/DeskVNCViewer.app/Contents/MacOS/` on macOS, in
`%LOCALAPPDATA%\DeskVNCViewer` or `C:\Program Files\DeskVNCViewer` on
Windows, and at `/usr/bin/dvv` on Linux. It adds an MCP entry to OpenCode's and
Codex's config, keeping a backup and any comments, and repoints one that names
an older `dvv`; writes the skill where each agent reads skills; and runs
`claude mcp add` for Claude Code.

### Pi, and other agents with no MCP client

Pi drives tools through its shell rather than MCP, so setup installs the skill
into `~/.pi/agent/skills` and puts `dvv` beside the `pi` command, which is on
PATH because that is how `pi` itself runs. The
agent then runs `dvv open`, `dvv screen`, `dvv click` and the rest as shell
commands. `dvv open` starts a small background holder so a machine stays open
between commands, and `dvv screen` saves the picture to a file and prints its
path, which Pi's `read` tool opens as an image.

### Free models

Desktops need a model that can see images. Text only models can still drive
an SSH terminal.

* In OpenCode, the free models that see images include
  `opencode/muse-spark-1.3-contributor-free`. Verified end to end: it opened
  a folder and a web page on a Windows machine with no help.
  On Windows it also opened Chrome on bible.com and read the verse of the
  day, and typed into a new Notepad tab and closed it without saving.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [psmux/DeskVNC](https://github.com/psmux/DeskVNC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
