---
trigger: always_on
description: A zsh plugin that adds Claude-powered helpers to the terminal:
---

# zsh-claude-code

A zsh plugin that adds Claude-powered helpers to the terminal:

- **`ask <question>`** — one-shot Q&A, prints a terse answer directly to the terminal.
- **`explain <command>`** — summarize a shell command in natural, concise English.
- **Ctrl+X widget** (`claude-suggest`) — rewrites the current command line (a natural-language request) into an executable shell command, in place. The user reviews and presses Enter to run (or edits first).
- **Alt+E widget** (`claude-explain-widget`) — explains the command currently typed at the prompt, printing the explanation above it while leaving the command intact.

All four use the `claude` CLI (Claude Code) in `--print` mode under the hood. The plugin is a thin, zero-runtime-dependency wrapper that:

1. Passes tuned `--system-prompt` text so Claude behaves like a terminal assistant.
2. Tunes flags for speed (`--no-session-persistence`, `--disable-slash-commands`, `--tools ""`).
3. Uses a cheaper/faster model (Haiku) for the inline-suggest widget, a prose-friendly model (Sonnet) for `explain`, and a more capable model (Opus) for `ask`.

## Target users

zsh users, typically with oh-my-zsh, who already use Claude Code and want low-friction terminal helpers without leaving their current shell/terminal/theme (pure, powerlevel10k, etc).

## Requirements

- `zsh` 5.0+
- The `claude` CLI installed and logged in (`claude login` or `ANTHROPIC_API_KEY` set)
- No other runtime dependencies

Dev-only tooling (managed via `mise`): `bats-core` (tests), `lefthook` + `commitlint` (commit-message linting). None of these are needed to use the plugin.

## Related projects

- [`HundredAcreStudio/zsh-claude`](https://github.com/HundredAcreStudio/zsh-claude) — talks to the Anthropic API directly (curl + jq), requires a personal API key. Different architecture.
- [`ArielTM/zsh-claude-code-shell`](https://github.com/ArielTM/zsh-claude-code-shell) — closest functional overlap. Also wraps the `claude` CLI, but triggers on `# <description>` + Enter (overrides `accept-line` globally) instead of a dedicated keybind. No `ask`/`explain` commands.

We chose a dedicated keybind over an Enter override because it is opt-in per keystroke and does not modify normal Enter behavior.

## Repository layout

```
zsh-claude-code/
├── CLAUDE.md                       # this file — context for Claude Code sessions
├── README.md                       # user-facing install & usage docs
├── LICENSE                         # MIT
├── mise.toml                       # pinned dev tools (bats, lefthook, commitlint)
├── lefthook.yml                    # commit-msg hook → commitlint
├── commitlint.config.js            # Conventional Commits rules
├── zsh-claude-code.plugin.zsh      # oh-my-zsh entrypoint: guard + sources lib/*
├── lib/
│   ├── common.zsh                  # env-var defaults + `_zsh_claude_code_run` helper
│   ├── ask.zsh                     # `ask` function + alias
│   ├── explain.zsh                 # `explain` function + alias
│   ├── suggest.zsh                 # Ctrl+X widget (`claude-suggest`)
│   └── explain-widget.zsh          # Alt+E widget (`claude-explain-widget`)
├── scripts/
│   ├── setup.zsh                   # unified contributor bootstrap (`mise run setup`)
│   └── doctor.zsh                  # env diagnostics (`mise run doctor`)
└── test/
    ├── smoke.zsh                   # end-to-end smoke
    └── *.bats                      # bats-core unit tests
```

## Public surface (env vars users can override)

Declared with `: ${VAR:=default}` so user exports in `.zshrc` **before** plugin load take precedence. (Models can be changed at runtime by re-exporting the var; keybinds cannot — the bind happens once at source time.)

- `ZSH_CLAUDE_ASK_MODEL` — default `sonnet`
- `ZSH_CLAUDE_EXPLAIN_MODEL` — default `sonnet`
- `ZSH_CLAUDE_SUGGEST_MODEL` — default `sonnet`
- `ZSH_CLAUDE_SUGGEST_KEY` — default `^X`
- `ZSH_CLAUDE_EXPLAIN_KEY` — default `^[e` (Alt+E; overrides the rarely-used `capitalize-word`)
- `ZSH_CLAUDE_ASK_SYSTEM_PROMPT` — full override of the `ask` system prompt
- `ZSH_CLAUDE_EXPLAIN_SYSTEM_PROMPT` — full override of the `explain` / explain-widget system prompt
- `ZSH_CLAUDE_SUGGEST_SYSTEM_PROMPT` — full override of the suggest-widget system prompt
- `ZSH_CLAUDE_EXTRA_FLAGS` — extra flags appended to every `claude` invocation (advanced)

Name prefixed `ZSH_CLAUDE_` to avoid collisions with the `claude` CLI's own env vars (`CLAUDE_*`, `ANTHROPIC_*`).

## Design notes / decisions

- **Why `--system-prompt` (full replace) instead of `--append-system-prompt`?** Claude Code's default system prompt is large and biased toward agentic tool-use. For quick terminal Q&A we don't want that context — full replace is faster and produces tighter answers.
- **Why `--tools ""`?** These helpers are text-in/text-out. Disabling tools prevents Claude from trying to read files or run commands, which would surprise the user and slow down responses.
- **Why not `--bare`?** `--bare` disables OAuth and the keychain, so users logged in via `claude login` would break. Stick to lighter flags that preserve auth.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matheus-poli/zsh-claude-code](https://github.com/matheus-poli/zsh-claude-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
