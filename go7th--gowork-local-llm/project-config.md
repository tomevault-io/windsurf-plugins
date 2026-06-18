---
trigger: always_on
description: Use the local `gowork` CLI (backed by MLX + Qwen3.5-4B) to offload token-heavy tasks (file summarization, data cleaning, text extraction, format conversion, bulk file processing) to a local LLM. Prefer `gowork --role summarize` which bundles the right model and flags automatically; fall back to `gowork --no-tools` if roles aren't configured. Invoke this skill whenever you need to process large files, summarize documents, clean messy data, extract structured info from unstructured text, or do any
---


# gowork Local LLM Offloader

Use the local `gowork` command to delegate token-intensive work to a free local model (MLX + Qwen3.5-4B-OptiQ-4bit), saving Claude API tokens for higher-value reasoning.

## Backend Architecture

```
gowork CLI (Rust, 30ms startup)
    │
    ▼
mlx_lm.server (:8080)
    │  OpenAI-compatible API
    ▼
Qwen3.5-4B-OptiQ-4bit (MLX, Apple Silicon optimized)
```

- **Backend**: `mlx_lm.server` running on `http://localhost:8080/v1`
- **Model**: `mlx-community/Qwen3.5-4B-OptiQ-4bit` (~3.4GB memory)
- **Hardware**: Apple M5 32GB, MLX framework (比 Ollama 快 20-50%)
- **Thinking mode**: 已关闭 (enable_thinking=false)，节省时间和内存

## When to Use gowork vs Claude

**Use gowork for** (mechanical, high-token tasks):
- Summarizing large files or documents
- Cleaning/formatting messy data (CSV, JSON, logs)
- Extracting specific fields from unstructured text
- Bulk text transformations across multiple files
- Getting a quick overview of a file's contents before deciding what to do
- Translating or reformatting text
- Generating boilerplate from templates

**Keep in Claude** (reasoning-heavy tasks):
- Architecture decisions, code review, debugging
- Multi-step planning that requires context from the conversation
- Tasks where quality matters more than token savings
- Anything that needs access to Claude's tools (file editing, git, etc.)

## Key Flags (Optimized for Preprocessing)

### Preferred: Use `--role summarize`

The user has a `summarize` role preset configured that already bundles `--no-tools` + the 4B model. **Always prefer this**:

```bash
# Simplest form — role takes care of --no-tools + model selection
gowork --role summarize --file big.go -p "list all functions"
gowork --role summarize -p "extract errors from this log"
```

For heavy code analysis (MoE 30B model), use `--role code`:
```bash
gowork --role code --file complex_module.rs -p "find bugs in this code"
```

For cloud fallback (when local is busy):
```bash
gowork --role deepseek --file data.json -p "clean and restructure"
```

Run `gowork --list-roles` to see all available roles on the user's system.

### Fallback: Manual `--no-tools`

If the user has no role config yet, fall back to `--no-tools` explicitly:

```bash
# Manual form (equivalent to --role summarize in role-aware setups)
gowork --no-tools -p "your prompt"
gowork --no-tools --file path/to/file.go -p "list all functions"

# Pipe stdin
cat file.txt | gowork --no-tools -p "summarize in 3 bullet points"
grep "^func " file.go | gowork --no-tools -p "list function names"

# Batch process multiple files (single process, reuses connection)
gowork --no-tools --batch "src/*.go" -p "one-line summary: {}"

# Cache results (skip re-processing unchanged files)
gowork --no-tools --cache --file data.csv -p "list column names"
```

### Full flag reference

| Flag | Purpose |
|------|---------|
| `-p "prompt"` | One-shot mode, runs prompt and exits |
| `--role <name>` | **Preferred.** Apply a named preset (`summarize`/`code`/`deepseek`/...). Bundles `--no-tools` + model + backend |
| `--list-roles` | Show all configured roles |
| `--no-tools` | Skip 8 tool definitions, 35% faster (implied by `--role summarize`) |
| `--file path` | Read file and prepend content to prompt |
| `--batch "glob"` | Process multiple files. `{}` = content placeholder |
| `--cache` | Cache by file path + mtime + prompt |
| `--stats` | Show token savings statistics |
| `--stats-reset` | Reset all statistics |
| `-m model` | Override model |
| `--base-url` | Override API URL |

## Direct curl Alternative (Zero Overhead)

For maximum speed, bypass gowork entirely and call MLX API directly:

```bash
curl -s http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d "{\"model\":\"mlx-community/Qwen3.5-4B-OptiQ-4bit\",\"messages\":[{\"role\":\"user\",\"content\":\"$(cat file.txt | head -100 | sed 's/"/\\"/g; s/$/\\n/' | tr -d '\n')summarize in 3 bullet points\"}],\"temperature\":0.1,\"max_tokens\":512}" \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['choices'][0]['message']['content'])"
```

Use curl when: you need absolute minimum latency, or gowork binary is unavailable.
Use gowork when: cleaner syntax matters, or you need --batch/--cache features.

## Environment Configuration

The following environment variables should already be set in the user's `~/.zshrc`:

```bash
export GOWORK_BASE_URL="http://localhost:8080/v1"
export GOWORK_MODEL="mlx-community/Qwen3.5-4B-OptiQ-4bit"
```

If gowork fails with "connection refused", the MLX server may not be running. Tell the user:
```bash
mlx-start   # alias to start mlx_lm.server
```

## Speed Optimization

### 原则一：先筛再喂（减少输入）

输入量直接决定速度。不要把整个文件丢给模型，先用 grep/head/awk 筛出关键内容：

```bash
# 慢 (18s)：把 500 行全喂给模型
gowork --no-tools --file bigfile.go -p "summarize"

# 快 (5s)：先 grep 关键行

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [go7th/gowork-local-llm](https://github.com/go7th/gowork-local-llm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
