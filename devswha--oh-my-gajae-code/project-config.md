---
trigger: always_on
description: Agent-facing guide for `oh-my-gajae-code`, a **plugin marketplace** for Gajae Code (`gjc`).
---

# AGENTS.md — working in oh-my-gajae-code

Agent-facing guide for `oh-my-gajae-code`, a **plugin marketplace** for Gajae Code (`gjc`).
Read this before adding or editing plugins. Human-facing intro lives in [README.md](./README.md).

## v0.28.0 identity cutover and migration

`oh-my-gajae-code` is the canonical repository, marketplace/plugin identity, and `./plugins/oh-my-gajae-code` source. `/omg:*` commands remain unchanged.

v0.27.0 was the final old-identity bridge. The canonical installer is `https://raw.githubusercontent.com/devswha/oh-my-gajae-code/main/install.sh`; old `raw.githubusercontent.com/devswha/oh-my-gjc/...` URLs do not redirect. Old GitHub repository pages and Git remotes redirect, but active instructions and local checkout names use `oh-my-gajae-code`.

New installs write only `oh-my-gajae-code` bindings. The former `oh-my-gjc` suite-root binding is a read-only fallback for at least 30 days or two releases; it is never rewritten or cleaned up by this cutover. Existing old XDG research data, credentials, and `models.yml` remain in place.

## What this repo is

A single git repo that catalogs installable `gjc` plugins. One `marketplace.json`
lists every plugin; each plugin is one directory under `plugins/`. The format is
compatible with the Claude Code / Codex plugin spec.

Plugins install from the **shell CLI** — `gjc plugin install <name>@<marketplace> …`
(TARGETS is plural: install several in one command; `--scope user` is the default,
`--scope project` pins to a repo; `gjc plugin marketplace add <ref>` registers a
catalog; `gjc plugin list` shows installed). **Plugin management is shell-CLI only — gjc has NO `/plugin` slash command** (verified against the core slash registry + live new-user repro 2026-07-08: `gjc plugin marketplace add`/`install`/`list` all rc=0). A `/plugin …` line typed inside a `gjc` session is just a chat message, not a command, so all install/uninstall/marketplace steps must run in a terminal. The registry lives at `~/.gjc/plugins/installed_plugins.json`. (`/plugin` slash is Claude-Code syntax — do NOT put it in gjc install docs.)

## Setup / Environment

### gjc
- Install gjc, then sign in to model providers via OAuth (Claude / OpenAI Codex / Kimi — no API key needed). Model presets:
  - `gjc --mpreset claude-max` — highest quality
  - `gjc --mpreset kimi` — cheaper worker / parallel
- **API keys** (web search, Gemini, etc.) must live in a **trusted location**, NOT the project `cwd/.env` (gjc ignores cwd `.env` for credentials). Copy the template and symlink it into your gjc home:
  ```sh
  cp .env.example .env                 # then fill in keys
  ln -sf "$(pwd)/.env" ~/.gjc/.env     # run once from the repo root
  ```
  Credential precedence: live env → `~/.gjc/agent/.env` → `~/.gjc/.env` → `~/.env`.
- **Web search:** `gjc config set providers.webSearch exa` (fallback: duckduckgo). Full key list (Exa/Tavily/Gemini/…) is in [`.env.example`](./.env.example).

### Capability prerequisites (single `oh-my-gajae-code` suite)
- `insane-review`: ChatGPT subscription + a Chromium-family browser on CDP `:9222` logged into chatgpt.com.
- `gpt-image`: POSIX deadline enforcement, the same logged-in dedicated ChatGPT Chromium CDP profile, and Python Playwright. It shares the CDP single-flight lease with `insane-review`; never run both concurrently.
- `insane-search`: Python 3, `curl_cffi>=0.15.0`, Beautiful Soup, PyYAML, and markdownify; `yt-dlp` is optional for supported media metadata/captions. It never installs dependencies automatically.
- `no-english`, `extragoal`, and the `example-plugin` template: no external prerequisites.

## Layout

```
oh-my-gajae-code/
├── .claude-plugin/
│   └── marketplace.json          # catalog: every plugin is registered here
├── plugins/
│   └── <plugin>/
│       ├── .claude-plugin/plugin.json   # manifest
│       ├── commands/<file>.md           # slash commands → /<plugin>:<file>  (generic convention — see note)
│       ├── agents/<file>.md             # sub-agents
│       ├── skills/<name>/SKILL.md       # skills
│       ├── hooks/hooks.json             # hooks
│       ├── .mcp.json                    # MCP servers
├── README.md                     # simple human intro
└── AGENTS.md                     # this file
```

> ⚠ `commands/` is the *generic* Claude-Code convention. In THIS repo the `oh-my-gajae-code` suite keeps its
> command bodies in `templates/` (a non-convention dir) because GJC 0.11 marketplace commands are
> exposed under the wrong `oh-my-gajae-code:*` namespace; `bin/install-skill.sh` installs `/omg:*` natively.

Content is discovered by **convention directories** above; explicit paths in
`plugin.json` are optional overrides.

## Add a plugin (procedure)

1. Create `plugins/<plugin>/.claude-plugin/plugin.json`.
2. Add content in convention dirs (`skills/<name>/SKILL.md`, `agents/`, `hooks/`, `.mcp.json`). Command bodies for the `oh-my-gajae-code` suite go in `templates/<name>.md` (NOT `commands/` — see the Layout note); a standalone plugin may use `commands/` but then gets the `<plugin>:<name>` namespace.
3. Register it in `.claude-plugin/marketplace.json` under `plugins`:
   ```json

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devswha/oh-my-gajae-code](https://github.com/devswha/oh-my-gajae-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
