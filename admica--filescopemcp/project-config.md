---
trigger: always_on
description: You are in the FileScopeMCP repository — an MCP server that indexes codebases for autonomous AI agents (Hermes, OpenClaw, Claude Code, Codex, and any other MCP-capable runtime).
---

# FileScopeMCP

You are in the FileScopeMCP repository — an MCP server that indexes codebases for autonomous AI agents (Hermes, OpenClaw, Claude Code, Codex, and any other MCP-capable runtime).

## What This Is

FileScopeMCP watches source code, ranks every file by importance (0-10), maps all import dependencies bidirectionally, extracts symbols (functions, classes, types) via tree-sitter, and maintains LLM-generated summaries. It exposes this intelligence as MCP tools so an agent can understand a codebase without reading every file.

Supports: TypeScript, JavaScript, Python, C, C++, Rust, Go, Ruby, Lua, Zig, PHP, C#, Java.

## If You Want to USE FileScopeMCP in Another Project

You don't need to be in this repo. Register it with your agent runtime, then point it at a project with `set_base_directory(path)` (or just launch the server from the project directory — it auto-initializes to its launch cwd).

### Stdio transport (all runtimes)

```
command: node
args: ["/path/to/FileScopeMCP/dist/mcp-server.js"]
```

### Hermes (`~/.hermes/config.yaml`)

```yaml
mcp_servers:
  filescope:
    command: "node"
    args: ["/path/to/FileScopeMCP/dist/mcp-server.js"]
    timeout: 120
    connect_timeout: 60
```

After editing, restart Hermes so it re-reads `mcp_servers`. Verify the server appears in Hermes's tool list and that `status()` returns `initialized: true`.

### Claude Code

Already registered if `./build.sh` was run. Verify with `claude mcp list`.

For an opinionated install that includes a project priming `CLAUDE.md` and pointers to optional hook templates: `npm run install-claude-code`. Layered — never modifies `.claude/settings.json`. See [ROADMAP.md](ROADMAP.md) Phase 1 for design notes and [docs/claude-code-hooks.md](docs/claude-code-hooks.md) for the hook snippets.

### Cursor / other MCP clients

See `docs/mcp-clients.md` for per-client config snippets.

## Connecting a Local LLM (Broker Setup)

FileScopeMCP runs a broker that queues LLM work (file summarization, change analysis). The broker talks to any OpenAI-compatible HTTP endpoint. Most agent runtimes already have a local LLM running — point the broker at the same endpoint and you avoid running two LLM servers.

The broker reads `~/.filescope/broker.json` (created automatically on first run from `broker.default.json`). Schema:

```json
{
  "llm": {
    "provider": "openai-compatible",
    "model": "llm-model",
    "baseURL": "http://localhost:8880/v1",
    "maxTokensPerCall": 1024
  },
  "jobTimeoutMs": 300000,
  "maxQueueSize": 1000
}
```

Set `baseURL` to wherever the LLM is listening, and `model` to whatever alias the server expects (check `/v1/models` if you're not sure). The broker handles queuing and prioritization — interactive tool calls beat background summarization, and it will not flood the LLM with hundreds of concurrent requests.

If `~/.filescope/broker.json` does not exist yet, copy a template:

```bash
mkdir -p ~/.filescope
cp broker.default.json ~/.filescope/broker.json
```

Then edit `baseURL` and `model` to match the runtime's LLM. Verify with `status()` — it reports broker connection state.

### Default: Linux-native (Hermes on Ubuntu)

This is the deployment FileScopeMCP is primarily designed for: agent runtime and llama-server on the same Ubuntu host, broker talking to `localhost`.

Typical Hermes setup already has llama-server bound to `127.0.0.1:8880` with model alias `llm-model` (Hermes uses it as `fallback_model` in `config.yaml`). The default broker template (`broker.default.json`) is configured for exactly this — copy it as-is:

```bash
cp broker.default.json ~/.filescope/broker.json
```

No edits needed if the runtime's llama-server is on `localhost:8880` with alias `llm-model`. To stand up a fresh llama-server, run `./setup-llm.sh` for a platform-aware install guide.

### Alternative: Remote LLM on the LAN

If the LLM runs on a separate machine reachable over the LAN (dedicated GPU box, etc.), use `broker.remote-lan.json` and edit the IP:

```bash
cp broker.remote-lan.json ~/.filescope/broker.json
# then edit baseURL to "http://<lan-ip>:8880/v1"
```

### Alternative: WSL2 with llama-server on the Windows host

Legacy setup for users running FileScopeMCP inside WSL2 with a GPU-equipped Windows host. WSL2 does not give native GPU access to llama.cpp, so llama-server runs on Windows and the broker bridges across the WSL2 boundary.

```bash
cp broker.windows-host.json ~/.filescope/broker.json
```

`broker.windows-host.json` uses the literal placeholder `wsl-host` in `baseURL`. At broker startup, `src/broker/config.ts` runs `ip route show default | awk '{print $3}'` and rewrites the placeholder to the actual Windows host gateway IP in memory — no manual editing needed in 99% of cases. See `docs/llm-setup.md` for the full Windows-side setup (firewall rule, llama-server install, model selection).

This path is *not* the default. If running on a native Linux host (including Hermes on Ubuntu), use `broker.default.json` instead.

## Skill File for Tool Usage


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [admica/FileScopeMCP](https://github.com/admica/FileScopeMCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
