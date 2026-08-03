---
trigger: always_on
description: - Repo lives on WSL2-mounted NTFS (OneDrive path at `/mnt/d/`) — run `git config --global --add safe.directory "/mnt/d/OneDrive - 上海达美乐比萨有限公司/documents/docs/code/go-code/MyProjects/git space/HackerTeam"` before any git commands
---

## Environment
- Repo lives on WSL2-mounted NTFS (OneDrive path at `/mnt/d/`) — run `git config --global --add safe.directory "/mnt/d/OneDrive - 上海达美乐比萨有限公司/documents/docs/code/go-code/MyProjects/git space/HackerTeam"` before any git commands
- `git` commands via Bash tool fail with "dubious ownership" without the safe.directory fix above
- `.go` files use CRLF line endings (Windows), `.md` files use LF — `Edit` tool fails on CRLF files, use `python3 -c "..."` via Bash instead
- `sed -i` fails with "Operation not permitted" on NTFS — write to `/tmp` and `cp` back, or use Python for in-place edits
- CRLF-safe edit for `.go` files: `python3 -c "import pathlib; p=pathlib.Path('file.go'); c=p.read_text(); c=c.replace('OLD','NEW'); p.write_text(c)"`

## Build & Run
- `go build -ldflags "-s -w" -o HackerTeam .` — build for current platform
- `./build.sh` (Linux) / `.\build.ps1` (Windows) — native build → `release/`
- `go run .` — run directly (auto-loads config from `<cwd>/.HackerTeam/`)
- `go vet ./...` — static analysis (passes clean)
- `go mod tidy` — sync dependencies after adding/removing imports
- **CGO required** — `memory/sqlite` depends on `mattn/go-sqlite3`. Cross-compilation no longer supported; build natively on each platform.
- Go module: `HackerTeam` (Go 1.26.1)

## Architecture
- Multi-agent AI pentesting platform: Captain serially dispatches Recon → Scanner → Exploit (cross-validate) → PostExploit, plus Reproducer in two batches (Batch1: after Scanner+Exploit, Batch2: after PostExploit)
- Each agent prompt must have a "职责边界" rule as the first constraint — explicitly list what this agent MUST NOT do and WHICH agent handles that; LLMs cross role boundaries unless explicitly forbidden (Recon may try sqlmap, Scanner may try to exploit). Forbidding tool NAMES is not enough — LLMs bypass "don't use sqlmap" by doing manual injection with the same payloads. Forbid concrete BEHAVIORS with exact examples (e.g. "NEVER append ' / OR 1=1 / UNION SELECT to URL params") so the LLM cannot self-rationalize. Do NOT add cross-boundary refusal logic on sub-agents — if Recon rejects and Scanner also rejects, tasks deadlock; enforce boundaries on Captain's dispatch side only, accept the risk of Captain hallucination.
- Shared consensus system in `global/prompts/common/` (embedded via `//go:embed` in `global/AgentEngine.go`): `vuln_consensus.md` (vulnerability definition + severity rating by technical impact, no CVSS), `output_consensus.md` (output format, raw tool output preservation, vulnerability structured block format for Reproducer consumption)
- TUI built with `rivo/tview` + `gdamore/tcell/v2`, PTY execution via `creack/pty`
- **TUI refactored (v1.0.0)**: `tui/tui.go` + `tui/tip/tip.go` merged into `global/TUI.go` + `global/tuihandler.go`. `handler/` and `bootstrap/` no longer import `tcell` directly — all TUI operations go through `global.PrintToTui()`, `global.LoadTextAreaWithEnter()`, etc. Old widget vars (`App_p`, `AgentMessageView_p`, `Sidebar_p` etc.) renamed to `app_p`, `AgentMessage`, `InputArea`, etc. The `tui/` directory no longer exists.
- **UI v1.1.0**: Sidebar removed; AgentPage layout is StatusBar + AgentMessage(no border, full flex) + InputRow(InputArea + `Ctrl+K 帮助` hint). Help page (`tview.Table`, two-column: command + description) shown via `app_p.SetRoot()` on Ctrl+K, dismissed with Esc/Ctrl+K, focus returns to InputArea.
- Agent framework: `trpc.group/trpc-go/trpc-agent-go`, MCP: `trpc.group/trpc-go/trpc-mcp-go`
- LLM backends: OpenAI-compatible API or Anthropic native SDK
- Config auto-generated at first run: `<binary-dir>/.HackerTeam/HackerTeam.yaml`
- TUI colors centralized in `utils/pretty/pretty.go` (TuiXxx constants)
- `/new`, `/flush`, `/exit`, `ESC` — built-in TUI commands
- Agent prompts embedded via `//go:embed` in `global/AgentEngine.go` (`PromptFiles`, `prompts/*` prefix in ReadFile) and `session/summarizer.go` (`promptFiles`, `prompt/*` prefix)
- Adding a new shared consensus prompt pattern: 1) create `global/prompts/common/<name>.md`, 2) add variable in `global/AgentEngine.go` + load in `Initializer.go` (follow `VulnConsensusPrompt` pattern), 3) add `{{<NAME>}}` replacement in `assemblePrompt()` in `members.go`, 4) add `{{<NAME>}}` placeholder to each agent prompt `.md` file
- `{{OUTPUTDIR}}` is the exception — NOT replaced by `assemblePrompt()`. It's resolved once in `env.md` via `configENVPrompt()` then injected into all agents through `{{ENV}}`. Agents infer the path from the "Output Directory" field shown in the environment block. Use `{{OUTPUTDIR}}` directly in prompt `.md` files, do NOT add Go-level replacement for it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lfz97/HackerTeam](https://github.com/lfz97/HackerTeam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
