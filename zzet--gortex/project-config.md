---
trigger: always_on
description: Code intelligence engine written in Go. Indexes repositories into an in-memory knowledge graph and exposes it via CLI and MCP Server.
---

# Gortex

Code intelligence engine written in Go. Indexes repositories into an in-memory knowledge graph and exposes it via CLI and MCP Server.

## Build & Test

```bash
go build -o gortex ./cmd/gortex/   # requires CGO (tree-sitter C bindings)
go test -race ./...                 # all test packages must pass
```

## Codebase Overview

- **Languages:** go (primary)
- **Entry point:** `cmd/gortex/main.go`
- **Source:** 1,338 Go files (728 non-test) across the `cmd/` and `internal/` trees
- **Graph size:** ~31k nodes, ~206k edges when the daemon indexes this repo

## MANDATORY: Use Gortex MCP tools instead of Read/Grep/Glob

Gortex is registered as an MCP server. You **MUST** prefer graph queries over file reads on every task in this repo — `search_symbols`, `find_usages`, `get_symbol_source`, `get_editing_context`, `smart_context`, `edit_symbol` / `edit_file` / `rename_symbol` / `batch_edit`. PreToolUse hooks deny `Read` / `Grep` / `Glob` against indexed source; the deny message names the right tool. The MCP server registers 120+ tools — all of them eagerly published in `tools/list` by default; set `GORTEX_LAZY_TOOLS=1` to opt into the lazy / `tools_search` discovery flow when the client honours `notifications/tools/list_changed`. The cross-project rule tables live in `~/.claude/CLAUDE.md` — neither is restated here. This file carries only project-specific guidance.

### Discovery (read once, then keep using)

- **Graph schema** — `gortex://schema` resource (node kinds, edge kinds, what each carries).
- **Analyzer rollups** — `gortex://report`, `gortex://surprises`, `gortex://god-nodes`, `gortex://questions`, `gortex://audit`.
- **Bootstrap state** — `gortex://stats`, `gortex://index-health`, `gortex://workspace`, `gortex://repos`, `gortex://active-project`.

### LLM provider (powers `ask` and `search_symbols assist:` modes)

Selected via `llm.provider` in `.gortex.yaml` or `~/.config/gortex/config.yaml`. The HTTP and subprocess providers are pure Go — available without `-tags llama`.

| Provider | Backend | Requires |
|---|---|---|
| `local` (default) | in-process llama.cpp | `-tags llama` build + `llm.local.model` (a `.gguf` path) |
| `anthropic` | Messages API | `llm.anthropic.model` + `ANTHROPIC_API_KEY` |
| `openai` | Chat Completions | `llm.openai.model` + `OPENAI_API_KEY`. Optional `llm.openai.effort` → `reasoning_effort`. |
| `azure` | Azure OpenAI Service | `llm.azure.deployment` + endpoint (`llm.azure.endpoint` or `AZURE_OPENAI_ENDPOINT`) + `AZURE_OPENAI_API_KEY`. Deployment-in-path + `api-version` query + `api-key` header; `llm.azure.api_version` defaults to a recent GA. Same json_schema structured output as `openai`. |
| `ollama` | Ollama daemon | `llm.ollama.model` (+ `llm.ollama.host`, default `localhost:11434`) |
| `claudecli` | `claude` CLI subprocess | `claude` on `$PATH` (signed in once); optional `llm.claudecli.model` (`sonnet`/`opus`/…). Reuses the user's Claude Code subscription. |
| `codex` | OpenAI `codex` CLI subprocess | `codex` on `$PATH` (signed in once); optional `llm.codex.model`. Runs `codex exec` in a read-only sandbox; reuses the user's Codex / ChatGPT sign-in. |
| `copilot` | GitHub Copilot CLI subprocess | `copilot` on `$PATH` (signed in via `gh`); optional `llm.copilot.model`. Runs `copilot -p`. |
| `cursor` | Cursor Agent CLI subprocess | `cursor-agent` on `$PATH` (signed in once); optional `llm.cursor.model`. Runs `cursor-agent --output-format text -p`. |
| `opencode` | opencode CLI subprocess | `opencode` on `$PATH` (signed in once); optional `llm.opencode.model` (`provider/model`). Runs `opencode run`. |
| `gemini` | Google Gemini `generateContent` REST | `llm.gemini.model` (default `gemini-2.5-pro`) + `GEMINI_API_KEY`. Structured output via `responseSchema` (`additionalProperties` stripped — Gemini rejects it). |
| `bedrock` | AWS Bedrock Converse API (SigV4) | `llm.bedrock.model_id` (e.g. `anthropic.claude-sonnet-4-20250514-v1:0`) + `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY` (+ optional `AWS_SESSION_TOKEN` for STS). Region defaults to `us-east-1` (`llm.bedrock.region`). Structured output via forced `respond` tool. No AWS SDK dependency — SigV4 is implemented in ~100 LOC of stdlib. |
| `deepseek` | DeepSeek Chat Completions (OpenAI-compatible) | `llm.deepseek.model` (default `deepseek-chat`) + `DEEPSEEK_API_KEY`. Structured output uses `response_format: json_object` plus a schema hint in the system prompt — DeepSeek does not support strict `json_schema`. |

`GORTEX_LLM_PROVIDER` / `GORTEX_LLM_MODEL` / `GORTEX_LLM_{CLAUDECLI,CODEX,COPILOT,CURSOR,OPENCODE}_BINARY` / `GORTEX_LLM_BEDROCK_REGION` / `GORTEX_LLM_AZURE_{ENDPOINT,DEPLOYMENT,API_VERSION}` / `GORTEX_LLM_EFFORT` override the file config. `GORTEX_LLM_MODEL` targets the active provider's model field (Gemini → `llm.gemini.model`, Bedrock → `llm.bedrock.model_id`, DeepSeek → `llm.deepseek.model`, etc.). If the active provider can't construct (missing model / API key, `local` without `-tags llama`, `claudecli` / `codex` without `claude` / `codex` on `$PATH`, `bedrock` without AWS credentials), the daemon logs a warning and `ask` stays unregistered — fall through to direct tools.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zzet/gortex](https://github.com/zzet/gortex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
