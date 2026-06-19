---
trigger: always_on
description: Pyramid-principle knowledge distillation — extract atoms, form groups, synthesize insights from journal notes.
---


# JS Knowledge Prism

A pyramid-principle-based knowledge distillation toolkit that transforms scattered journal notes into structured knowledge outputs.

## First Step: Detect Runtime Mode

Before performing any operation, detect whether this project is running as an **OpenClaw plugin** or in **standalone CLI mode**. The result determines configuration paths, command prefixes, and available features.

### Detection Steps

#### Step 0 — OS & Environment Variable Probe

First detect the current operating system to choose the correct shell commands, then check OpenClaw-related environment variables:

**OS Detection:**

| Check | Windows | macOS / Linux |
|-------|---------|---------------|
| OS identification | `echo %OS%` or `$env:OS` (PowerShell) | `uname -s` |
| Home directory | `%USERPROFILE%` | `$HOME` |
| Default OpenClaw state dir | `%USERPROFILE%\.openclaw\` | `~/.openclaw/` |
| Default config path | `%USERPROFILE%\.openclaw\openclaw.json` | `~/.openclaw/openclaw.json` |

**Environment Variable Check:**

```bash
# Windows (PowerShell)
Get-ChildItem Env: | Where-Object { $_.Name -match '^OPENCLAW_' }

# Windows (CMD / Git Bash)
set | grep -iE "^OPENCLAW_"

# macOS / Linux
env | grep -iE "^OPENCLAW_"
```

| Variable | Meaning if set |
|----------|---------------|
| `OPENCLAW_CONFIG_PATH` | Direct path to config file (e.g. `D:\.openclaw\openclaw.json`) — **highest priority**, use as-is |
| `OPENCLAW_STATE_DIR` | OpenClaw state directory (e.g. `D:\.openclaw`) — config file at `$OPENCLAW_STATE_DIR/openclaw.json` |
| `OPENCLAW_HOME` | Custom home directory (e.g. `D:\`) — state dir resolves to `$OPENCLAW_HOME/.openclaw/` |

**OpenClaw config file resolution order** (first match wins):

1. `OPENCLAW_CONFIG_PATH` is set → use that file directly
2. `OPENCLAW_STATE_DIR` is set → `$OPENCLAW_STATE_DIR/openclaw.json`
3. `OPENCLAW_HOME` is set → `$OPENCLAW_HOME/.openclaw/openclaw.json`
4. None set → default `~/.openclaw/openclaw.json` (Windows: `%USERPROFILE%\.openclaw\openclaw.json`)

Use the resolved config path in all subsequent steps.

#### Step 1 — OpenClaw Binary Detection

1. Check if `openclaw` command exists on PATH (Windows: `where openclaw`, macOS/Linux: `which openclaw`)
2. If exists, read the OpenClaw config file (path resolved by Step 0) and look for `js-knowledge-prism` in `plugins.entries` with `enabled: true`
3. Verify that `plugins.load.paths` contains a path pointing to this project's `openclaw-plugin/` directory

If **all three checks pass** → use **OpenClaw Plugin Mode**. Otherwise → use **Standalone CLI Mode**.

### Mode Comparison

| Aspect | OpenClaw Plugin Mode | Standalone CLI Mode |
|--------|---------------------|-------------------|
| Configuration | `~/.openclaw/openclaw.json` → `plugins.entries.js-knowledge-prism.config` | `.knowledgeprism.json` + `.env` |
| Command prefix | `openclaw prism <cmd>` | `npx js-knowledge-prism <cmd>` |
| AI tools | `knowledge_prism_*` (18 tools via OpenClaw Agent) | Not available (use CLI) |
| Cron auto-processing | `openclaw prism setup-cron` / `setup-output-cron` | Not available |
| Register / batch | `knowledge_prism_register` / `process_all` / `output_all` | Not available |
| Web UI | `http://<host>/plugins/js-knowledge/prism/` | Not available |
| Runtime data | `.openclaw/prism-processor/` (registry, inbox, batch) | None |
| Memory sync | Automatic to `work_dir/memory-export/` | Not available |

### OpenClaw Plugin Mode

When the plugin is deployed:

- **CLI**: always use `openclaw prism ...` instead of `npx js-knowledge-prism ...`
- **AI tools**: prefer `knowledge_prism_*` tools when invoked from an OpenClaw Agent session
- **Config**: modify `~/.openclaw/openclaw.json` for API endpoints, model, cron intervals, etc.; do NOT edit `.knowledgeprism.json` for plugin-managed settings
- **Cron**: manage via `openclaw prism setup-cron` / `openclaw prism setup-output-cron`
- **Registration**: use `openclaw prism register <dir>` to add knowledge bases for batch auto-processing
- **Output bindings**: use `knowledge_prism_bind_output` to configure automatic output generation
- **Runtime data**: check `.openclaw/prism-processor/registry.json` for registered bases and binding state

### Standalone CLI Mode

When running without OpenClaw:

- **CLI**: use `npx js-knowledge-prism <cmd>`
- **Config**: `.knowledgeprism.json` for processing params, `.env` for API credentials (see environment variable table in README)
- **No cron / register / batch** features — run `process` and `output` manually
- **No AI tools** — all interaction through CLI commands

---

## Deployment Probe

After detecting the runtime mode, run the following diagnostic steps to build a complete picture of the local deployment. Execute these in order; skip remaining steps if an earlier step indicates OpenClaw is unavailable.

> **Prerequisite**: Step 0 (OS & Environment Variable Probe) from the Detection Steps above must have already been executed. Use the detected OS to choose correct commands, and use the resolved config path from Step 0.

### Step 1 — OpenClaw Availability

- Windows: `where openclaw` / macOS & Linux: `which openclaw`
- If found: `openclaw --version` to confirm the installed version

### Step 2 — Plugin Load Status


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imjszhang/js-knowledge-prism](https://github.com/imjszhang/js-knowledge-prism) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
