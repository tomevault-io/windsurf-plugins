---
trigger: always_on
description: This section documents the headless/Docker characteristics of major AI coding CLI agents. Claude Code and Codex are supported in v1; additional agents are researched for future versions.
---

# AI Coding Agents

## AI Coding CLI Agents: Multi-Agent Support Research

This section documents the headless/Docker characteristics of major AI coding CLI agents. Claude Code and Codex are supported in v1; additional agents are researched for future versions.

**Resolved research gaps (v1):**
- **Codex proxy support:** Not supported. The static Rust binary does not honor `HTTP_PROXY`/`HTTPS_PROXY`. Open upstream issues: github.com/openai/codex#4242, github.com/openai/codex#6060.
- **Codex required network domains:** Confirmed `api.openai.com` only. Telemetry uses user-configured OTLP endpoints, not hardcoded domains.
- **Codex TUI behavior in tmux:** Confirmed working. Interactive mode runs correctly in tmux.

### Viable Agents

#### Claude Code

- **Install:** `npm i -g @anthropic-ai/claude-code`
- **Headless command:** `claude --dangerously-skip-permissions -p "task"`
- **API key env vars:** `ANTHROPIC_API_KEY`
- **State dir:** `~/.claude/`
- **Model selection:** `--model <model>`
- **Sandbox bypass:** `--dangerously-skip-permissions`
- **Runtime:** Node.js
- **Root restriction:** Refuses to run as root
- **Docker quirks:** Requires tmux with double-Enter workaround to submit prompts; needs non-root user

##### Claude Code Streaming Protocol (stream-json mode)

Claude Code supports a machine-readable streaming mode that enables programmatic integration without a TTY or tmux. This is separate from yoloai's current interactive mode but relevant to future headless/programmatic scenarios.

**Invocation:**
```
claude --output-format stream-json --input-format stream-json --verbose --model <model> [--session-id <id>] --dangerously-skip-permissions
```

**Communication:** Newline-delimited JSON on stdin/stdout. The process stays alive across multiple turns — subsequent user messages are written to the same stdin. A new process spawn is only needed when the session is explicitly ended or the process exits.

**Input format** (written to stdin per turn):
```json
{"type":"user","message":{"role":"user","content":[{"type":"text","text":"..."}]}}
```

**Output message types:**
| Type | Meaning |
|------|---------|
| `system` / `init` | Process startup; carries initial `session_id` |
| `content_block_start` | Start of text/thinking/tool_use block (keyed by `index`) |
| `content_block_delta` | Streaming delta: `text_delta`, `thinking_delta`, `input_json_delta` |
| `content_block_stop` | Block finished |
| `assistant` | Complete non-streaming assistant message |
| `user` | Tool results fed back from Claude Code |
| `result` | **End of turn** (see below) |

**The `result` message is the authoritative idle/done signal:**
```json
{
  "type": "result",
  "session_id": "abc123",
  "is_error": false,
  "total_cost_usd": 0.012,
  "duration_ms": 4200,
  "duration_api_ms": 3100,
  "num_turns": 3,
  "usage": {"input_tokens": 1234, "output_tokens": 567}
}
```
`is_error: true` indicates the turn failed. This is cleaner than timeout or output-stability heuristics for detecting completion.

**Session resumption via `--session-id`:** The `session_id` from the `result` or `system/init` message can be saved and passed back as `--session-id` on the next invocation. This allows Claude Code to resume its conversation context after a process restart or container recreation — the session history is stored in Claude Code's own state (`~/.claude/`), so as long as `agent-state/` is preserved, the session can be resumed.

**Environment requirement:** `TERM=xterm-256color` must be set in the subprocess environment; omitting it causes Claude CLI to misbehave.

**MCP tools in stream-json mode:** MCP server tools appear in the output stream with name format `mcp__<server>__<tool>`. Claude Code runs its configured MCP servers internally (from `~/.claude/settings.json`); they are transparent to the caller.

**Source:** Analysis of [opencode-claude-code-plugin](https://github.com/unixfox/opencode-claude-code-plugin) (2026-03), which implements the Vercel AI SDK `LanguageModelV2` interface on top of `claude` subprocess stdio.

#### OpenAI Codex

- **Install:** Static binary download or `npm i -g @openai/codex`
- **Headless command:** `codex exec --yolo "task"`
- **API key env vars:** `CODEX_API_KEY` (preferred), `OPENAI_API_KEY` (fallback)
- **State dir:** `~/.codex/`
- **Model selection:** `--model <model>` or `-m <model>` (e.g., `gpt-5.3-codex`, `gpt-5.3-codex-spark`, `codex-mini-latest`)
- **Model aliases:** `default` → `gpt-5.3-codex`, `spark` → `gpt-5.3-codex-spark`, `mini` → `codex-mini-latest`
- **Sandbox bypass:** `--yolo` (alias `--dangerously-bypass-approvals-and-sandbox`)
- **Runtime:** Rust (statically-linked musl binary, zero runtime deps)
- **Proxy support:** Not supported — does not honor `HTTP_PROXY`/`HTTPS_PROXY` (upstream issues github.com/openai/codex#4242, #6060)
- **Root restriction:** None found, but convention is non-root
- **Docker quirks:** `codex exec` avoids TUI entirely — no tmux needed; `--skip-git-repo-check` useful outside repos; Landlock sandbox may fail in containers (use `--yolo`); TUI works correctly in tmux

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kstenerud/yoloai](https://github.com/kstenerud/yoloai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
