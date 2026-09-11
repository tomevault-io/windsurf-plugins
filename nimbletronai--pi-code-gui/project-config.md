---
trigger: always_on
description: > AGENTS.md is loaded by Pi at startup. It covers project-specific
---

# Project Protocol

> AGENTS.md is loaded by Pi at startup. It covers project-specific
> operating conventions. Behavioral rules (how to think, how to verify,
> how to research) live in `.pi/APPEND_SYSTEM.md`. Detailed protocols
> live in `agent-wiki/` — this file is the map, not the territory.

## Project overview

Pi Code Gui is a VS Code extension that embeds the Pi coding agent into VS Code's
native UI. It provides a webview-based chat panel, 17 bridge tools that give the
AI agent access to VS Code editor state (diagnostics, symbols, hover, definitions,
references, and workspace edits), a sidebar with session and package tree views,
multi-session tab support with per-session model/thinking settings, and session
persistence via Pi's standard `.jsonl` format.

Key concepts:
- **SessionWindow** (`src/extension.ts`) — paired PiService + PiWebviewPanel, one per chat tab
- **PiService** (`src/pi-service.ts`) — SDK lifecycle bridge, event translation, model/settings
- **PiWebviewPanel** (`src/webview-panel.ts`) — webview creation, bidirectional messaging, rendering
- **Bridge Tools** (`src/bridge-tools.ts`) — 17 VS Code API tools for the AI agent
- **Event Translation** (`src/pi-service.ts` handleAgentEvent) — SDK events → webview messages
- **Extension UI Bridge** (`src/pi-service.ts` bindExtensionUI) — TUI widgets → webview
- **Webview Frontend** (`media/`) — chat UI, morphdom streaming, marked rendering
- **Tree Views** (`src/extension.ts` MultiSessionTreeProvider, `src/pi-packages-tree-provider.ts`)
- **Runtime Selection** (`src/pi-service.ts` `_backendKind`; `src/rust-*.ts`; `src/runtime-detection.ts`) — per-session TypeScript (in-process SDK) or Rust (out-of-process `pi --mode rpc`) Pi; default TypeScript; see `agent-wiki/architecture/runtime-selection.md`

See `agent-wiki/index.md` for the full topic catalog.

## Development workflow

```bash
pnpm install          # Install dev dependencies
pnpm run compile      # Type-check (tsc --noEmit), lint (eslint src), build (esbuild)
pnpm run watch        # Watch mode — esbuild + tsc in parallel
pnpm run package      # Production build (type-check + lint + minified esbuild)
pnpm run check-types  # Type-check only
pnpm run lint         # ESLint only
pnpm test             # Run tests via vscode-test
```

**Local dev loop:** Run `pnpm run watch`, then press `F5` in VS Code to launch
the Extension Development Host with live rebuilds.

**Packaging:** `pnpm run vsix` creates `pi-code-gui-x.x.x.vsix`.
Install locally: `code --install-extension pi-code-gui-*.vsix --force`.

**CI/CD:** GitHub Actions (`publish.yml`) triggers on GitHub Release.
Publishes to VS Code Marketplace (`vsce publish`) and Open VSX (`ovsx publish`).
Requires the `marketplace` environment with reviewer gates.

## pi clean-room — license law

Do not read, fetch, paste, or reference the source of `pi_agent_rust` and its restricted
runtime deps `asupersync`, `franken-decision`, `franken-evidence`, `franken-kernel` into
any agent context — **by any channel**: the Read tool, Bash (`cat`/`grep`/`sed`/`git
show`/`git log -p` on a checkout), or WebFetch of the repo's source/blob/raw/commit
pages. That covers local clones (e.g. a `~/pi_agent_rust` checkout), the `~/.cargo`
checkout/registry copies, and fork diffs (a fork's diff of restricted source is a
derivative work — same rule). These crates ship under "MIT + OpenAI/Anthropic Rider":
the Software and derivatives may not be made available to a Restricted Party (OpenAI,
Anthropic, their affiliates/agents), and Claude Code is an Anthropic surface — content
read into it is provided to Anthropic.

Work black-box instead: drive `pi --mode rpc` and capture its stdout (wire probes are
the established pattern), read its version/`--help` output, read its GitHub *issues*
(prose), and keep our own API notes in `agent-wiki/`. Reading the binary's observable
behavior is fine; reading its source is not.

`github.com/earendil-works/pi` (plain MIT © 2025 Mario Zechner) is the ancestor
`pi_agent_rust` was ported from. It carries no rider and MAY be read and fetched freely
as a clean-room reference. If you port ancestor code verbatim, carry its LICENSE note in
the module header + a NOTICE entry.

Enforced, not just prose: `permissions.deny` Read()/WebFetch rules + a Bash PreToolUse
hook in `.claude/settings.json`, and `scripts/check-cleanroom.sh` (+ `-smoke.sh`) at
pre-commit (`.githooks/`), `pretest`, and `package`. Removing a deny rule fails the
commit.

## Tool discipline

Raw bash for dev-loop (build, test, lint). VS Code extension publishing is
handled by CI.

**File edits should use `vscode_apply_workspace_edit`** (via the edit/write
tools) to keep open editor buffers in sync with disk. Direct file writes
bypass VS Code's buffer tracking and cause dirty-state mismatches.

## Storage rules

**Durable (versioned in git):** `AGENTS.md`, `agent-wiki/`, source in `src/`,
`media/` assets, config files at repo root.

**Durable (on disk, not versioned):**
- Pi sessions: `.jsonl` files in `~/.pi/agent/sessions/` or custom
  `pi-code-gui.sessionDir`. Survive VS Code restarts. Appear in Past Sessions.
- Pi packages: installed to `.pi/npm/node_modules/` (project) or global npm.
  The `.pi/` directory is gitignored.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NimbleTronAI/pi-code-gui](https://github.com/NimbleTronAI/pi-code-gui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
