---
trigger: always_on
description: agentmask is an open-source secrets firewall for AI coding agents. It prevents Claude Code, Cursor, and other AI assistants from reading, leaking, or committing secrets like API keys, tokens, passwords, and connection strings.
---

# agentmask — Contributor Context

## What This Is

agentmask is an open-source secrets firewall for AI coding agents. It prevents Claude Code, Cursor, and other AI assistants from reading, leaking, or committing secrets like API keys, tokens, passwords, and connection strings.

**agentmask's value is the IDE integration layer** — hooks, blocklist, MCP server, behavioral rules. Currently supports Claude Code and Cursor with auto-detection. Primary detection is delegated to [gitleaks](https://github.com/gitleaks/gitleaks) (150+ battle-tested rules). An **agentmask scanner** (`src/scanner/tier2.ts`) runs as a second pass to catch patterns gitleaks's `generic-api-key` rule deliberately excludes: `password`/`passwd`/`pwd` field assignments, connection strings with embedded credentials, and provider prefixes without dedicated gitleaks rules (`whsec_`, `GOCSPX-`).

## Architecture

```
agentmask = IDE integration layer (Claude Code + Cursor)
  ├── Scanner backend: gitleaks (150+ rules, auto-downloaded if not installed)
  ├── agentmask scanner: complementary TS regex pass (password fields, conn strings, whsec_, GOCSPX-)
  ├── Blocklist manager (.agentmask/blocklist.json, shared between IDEs)
  ├── IDE adapters (src/ide/targets.ts — hook format, rules format, config paths per IDE)
  ├── Hooks (pre-read, pre-write, pre-bash, post-scan) with --format adapter
  ├── MCP server (safe_read, env_names, scan_file, scan_staged)
  └── Behavioral rules (per-IDE: .claude/rules/*.md, .cursor/rules/*.mdc)
```

### Three Reinforcing Layers

```
Layer 1: BLOCK (PreToolUse hooks)
  → Pre-read: static path patterns + dynamic blocklist lookup (no subprocess, <5ms)
  → Pre-write: gitleaks + agentmask scan on content via temp file (~200ms)
  → Pre-bash: pattern match on commands + gitleaks scan on staged files for git commit
  → Post-scan: gitleaks + agentmask scan on tool output, warns + auto-adds to blocklist

Layer 2: REDIRECT (MCP server)
  → safe_read: reads file, uses merged gitleaks + agentmask findings to redact secrets
  → env_names: lists .env variable names without values
  → scan_file / scan_staged: explicit scans (scan_file also runs agentmask scanner)

Layer 3: INSTRUCT (.claude/rules/agentmask.md)
  → Behavioral rules telling the agent to prefer safe_read
  → Installed automatically by `agentmask init`
```

## The Blocklist System

The key innovation is the **dynamic blocklist** (`.agentmask/blocklist.json`):

1. `agentmask init` runs `gitleaks dir .` on the entire repo, then runs the agentmask scanner on the same tree, and merges the findings
2. Every file containing a detected secret (from either scanner) is added to the blocklist
3. Pre-read hook checks the blocklist on every Read call — blocked files never enter context
4. Post-scan hook catches secrets in files NOT in the blocklist (new/modified files) and auto-adds them
5. First read of a new secret file still leaks (unavoidable), but every subsequent read is blocked
6. `allow-path` removes entries from the blocklist (after secrets are fixed)
7. Re-running `agentmask init` rescans and rebuilds the blocklist

## Project Structure

```
src/
├── cli.ts                  # CLI entrypoint — Commander.js, 8 commands
├── cli/
│   ├── scan.ts             # `agentmask scan` — gitleaks + agentmask scanner, merged output
│   ├── init.ts             # `agentmask init` — gitleaks + agentmask scan + blocklist + hooks + MCP + rules
│   ├── remove.ts           # `agentmask remove` — clean uninstall including blocklist
│   └── allowlist.ts        # `allow-path` (also removes from blocklist), `allow-value`
├── gitleaks/
│   ├── binary.ts           # Find system gitleaks or auto-download from GitHub releases
│   ├── runner.ts           # Subprocess wrapper: scanDir, scanFile, scanContent, scanStaged
│   └── index.ts            # Barrel export
├── scanner/
│   ├── file-patterns.ts    # Static blocked path globs (.env, *.pem, etc.), binary detection
│   └── tier2.ts            # agentmask scanner: password fields, conn strings, whsec_, GOCSPX-
├── ide/
│   └── targets.ts          # IDE adapters: Claude Code + Cursor (hook format, rules, config paths, detection)
├── hooks/
│   ├── common.ts           # Hook I/O: readStdin, block(), allow(), --format adapter, safety timer
│   ├── blocklist.ts        # Dynamic blocklist (.agentmask/blocklist.json): load, save, query, migrate
│   ├── pre-read.ts         # Static patterns + blocklist lookup → block or allow (no subprocess)
│   ├── pre-bash.ts         # Command pattern match + gitleaks scanStaged on git commit
│   ├── pre-write.ts        # gitleaks + agentmask scanContent on content being written
│   └── post-scan.ts        # gitleaks + agentmask scanContent on tool output, warns + auto-blocklists
├── mcp/
│   └── server.ts           # MCP server with 4 tools (uses @modelcontextprotocol/sdk)
└── config/
    ├── loader.ts           # .agentmask.toml config loader + merging
    └── index.ts
```

## Key Design Decisions

- **gitleaks as the primary scanner** — we don't reinvent detection. gitleaks has 150+ battle-tested rules. Auto-downloaded if not installed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adithyan-ak/agentmask](https://github.com/adithyan-ak/agentmask) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
