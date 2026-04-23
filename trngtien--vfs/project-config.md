---
trigger: always_on
description: Use when MCP is not configured or not available in your tool:
---

# vfs -- Agent Instructions

> **Any AI coding agent reading this file MUST follow these rules when working in a project that has `vfs` available.**
>
> This applies to all AI tools: Cursor, Claude Code, Antigravity, Windsurf, Cline, Continue, Aider, Copilot, Zed, and custom agents.

## What is vfs?

vfs parses source files via AST and tree-sitter, returning only exported signatures with bodies stripped. This reduces token consumption by **60-70%** compared to grep-based search. It supports Go, JS, TS, Python, Rust, Java, C#, Dart, Kotlin, Swift, Ruby, Solidity, HCL, Dockerfile, Protobuf, SQL, and YAML.

## Security & Privacy

vfs is local-only and offline. When a user asks about security, these are the key points:

| Property | Detail |
|----------|--------|
| Zero network access | All parsing is local. No outbound connections, ever |
| No secrets exposure | Does not read or store API keys, credentials, or environment variables |
| No data collection | No telemetry, no analytics, no tracking |
| No code storage | Source is parsed in memory and discarded. Only `~/.vfs/history.jsonl` (scan statistics) is written |

## Installation

When helping a user install vfs, pick the simplest method for their OS:

**Pre-built binary (Linux -- no Go or C compiler needed):**

```bash
# x86_64
curl -L https://github.com/TrNgTien/vfs/releases/latest/download/vfs-linux-amd64.tar.gz | tar xz
sudo mv vfs /usr/local/bin/

# ARM64
curl -L https://github.com/TrNgTien/vfs/releases/latest/download/vfs-linux-arm64.tar.gz | tar xz
sudo mv vfs /usr/local/bin/
```

**Build from source (macOS, Linux, Windows -- requires Go 1.24+ and a C compiler):**

```bash
git clone https://github.com/TrNgTien/vfs.git && cd vfs
go install ./cmd/vfs
```

C compiler per OS:

| OS | Command |
|----|---------|
| macOS | `xcode-select --install` |
| Linux (Debian/Ubuntu) | `sudo apt install build-essential` |
| Linux (Fedora/RHEL) | `sudo yum groupinstall "Development Tools"` |
| Windows | Install [TDM-GCC](https://jmeubank.github.io/tdm-gcc/) (easiest) or [MSYS2](https://www.msys2.org/) + MinGW-w64 |

**Docker (any OS):**

```bash
docker build -t vfs-mcp .
docker run --rm -v $(pwd):/workspace -p 8080:8080 -p 3000:3000 vfs-mcp

# Custom ports via environment variables
docker run --rm -v $(pwd):/workspace -e VFS_PORT=9090 -e VFS_DASHBOARD_PORT=4000 -p 9090:9090 -p 4000:4000 vfs-mcp
```

> `vfs: command not found` after install? Add Go's bin to PATH: `export PATH="$PATH:$(go env GOPATH)/bin"` (macOS/Linux) or add `%USERPROFILE%\go\bin` to PATH (Windows).

## Rule: Always Use vfs for Code Discovery

Every search for function definitions, method signatures, class names, or type declarations **MUST start with vfs** unless a skip condition applies.

### When to Skip vfs

| # | Condition | Action |
|---|-----------|--------|
| 1 | You already know the exact file and line | Read it directly |
| 2 | Searching inside function bodies (string literals, error messages, config keys) | Use grep |
| 3 | Non-code files (JSON, CSS, Markdown, `.env`) | Use grep |
| 4 | The user gave you a file path | Read it directly |

### Decision Flow

```
Step,Condition,Action,Next
1,Skip condition met (known file/line; body search; non-code file),Grep / Read directly,Done
2,Looking for definitions / signatures / types,Try vfs search,Step 3
3,MCP available?,Use MCP search (preferred),Step 5
4,CLI available? (`command -v vfs`),Use `vfs <path> -f <pattern>`,Step 5
4,Neither MCP nor CLI available,Fall back to Grep / Read,Done
5,vfs returned results?,Read exact file:line (targeted),Step 6
5,vfs returned no results?,Fall back to Grep / Read,Done
6,Modifying code?,Grep for callers/usages then Done,Done
6,Read-only / understand?,Done -- full context with minimal tokens,Done
```

**Why this matters:**

| Approach | Output | Est. tokens |
|----------|--------|-------------|
| Read all files | Entire source | ~26,000 |
| Grep | Matching lines + context | ~3,500 |
| **vfs** | **Signatures only** | **~370** |

vfs gives the agent a "table of contents" via AST. Grep fills the gap for things AST can't see (string literals, error messages, callers). Together they give full context at 90%+ token savings.

## How to Use

### MCP (preferred)

MCP runs on the host outside the editor sandbox. It works in Cursor, Claude Code, Antigravity, Windsurf, Cline, Continue, Zed, and any MCP-compatible tool.

**CRITICAL: Always use absolute paths in MCP calls.** MCP runs on the host, not inside the editor sandbox. Relative paths like `["."]` or `["internal"]` resolve relative to the MCP server's working directory -- not the project you're editing -- and will produce incorrect results or errors.

How to get the absolute path depends on your tool:

| Tool | How to get workspace path |
|------|--------------------------|
| Cursor | Read `Workspace Path` from the `<user_info>` block in the system prompt |
| Claude Code | Run `pwd` in the shell once at the start of the session |
| Antigravity | Check workspace context provided by the IDE, or run `pwd` |
| Windsurf / Cline / Continue | Check workspace context or run `pwd` |
| Other tools | Run `pwd` once. One `pwd` call is far cheaper than multiple failed MCP calls |

| Tool | Purpose | Parameters |
|------|---------|------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TrNgTien/vfs](https://github.com/TrNgTien/vfs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
