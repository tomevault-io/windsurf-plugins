---
trigger: always_on
description: You are one of: the plan agent, a prover agent, a subagent (per descriptor in `.archon/subagents/`), or the review agent. Read `PROGRESS.md` to determine your role and current objectives. Keep workspace tidy. Prefer existing MCP tools.
---

# Archon Project

You are one of: the plan agent, a prover agent, a subagent (per descriptor in `.archon/subagents/`), or the review agent. Read `PROGRESS.md` to determine your role and current objectives. Keep workspace tidy. Prefer existing MCP tools.

## Priority Rule

When instructions conflict between global and local sources, **local takes precedence**:

- Prompts in `.archon/prompts/` override Archon's global prompts.
- Under Claude Code, skills in `.claude/skills/` override globally installed plugins.
- Under Claude Code, rules in `.claude/rules/` apply only to this project.

## Skills

- **archon-lean4** — under Claude Code this is installed as the `lean4@archon-local` plugin (live-linked to Archon source), providing slash commands `/archon-lean4:prove`, `/archon-lean4:golf`, `/archon-lean4:doctor`, etc. Under other harnesses the slash-command surface is absent; the underlying skill scripts are still on disk under `.claude/skills/` and can be run directly from the shell.

## Tools

Project tools live in `.claude/tools/` as directly-executable scripts — that path is the same whatever harness you run under. List with `ls .claude/tools/`; run `<path> --help`. MCP servers (Lean LSP, etc.) surface as `mcp__*` tools: under Claude Code they are registered in `.claude/settings.json` / `.mcp.json`; under other harnesses the loop wires the same servers in at launch.

Two always-present scripts:

- **`archon-informal-agent.py`** — call external LLMs for an informal proof-style sketch when you're stuck on an approach and want a second opinion. Supports DeepSeek, Kimi (OpenAI-compatible via `--provider kimi`), Kimi (Anthropic-compatible via `--provider kimi-anthropic`), OpenRouter, OpenAI, and Gemini; defaults to `--provider auto` which picks the best available key automatically. **Requires at least one key in env** (`DEEPSEEK_API_KEY` / `MOONSHOT_API_KEY` / `OPENROUTER_API_KEY` / `OPENAI_API_KEY` / `GEMINI_API_KEY`) — check via `env | grep -E "DEEPSEEK|MOONSHOT|OPENROUTER|OPENAI|GEMINI"` BEFORE planning around its use; if no key is set, fall back to alternatives below. Both `kimi` and `kimi-anthropic` use the same `MOONSHOT_API_KEY`; the latter speaks Moonshot's Anthropic-compatible Messages endpoint and supports `--think`. The output is LLM-generated from training data, NOT source-derived: this tool is **not** a literature retriever. For actual literature lookup consult the auto-injected subagent catalog — when a literature/reference-fetching subagent is enabled it will appear there — or use the `WebSearch` / `WebFetch` tools directly.
- **`archon-subagent.py`** — the generic subagent dispatcher (one wrapper handles every subagent — see "Subagents" below).

## Subagents

Subagents are descriptor files at `.archon/subagents/<name>.md` (YAML frontmatter — `name`, `description`, `write_domain`, `read_only`, `can_spawn`, `default_enabled`, optional `mandatory: [<phase>...]`, optional `dispatcher_notes` — followed by the prompt body the spawned agent reads).

**You do NOT need to discover subagents.** The plan and review prompts auto-inject an **Available subagents** section at the top of each invocation listing every enabled descriptor with description, write-domain, MANDATORY flag, and `dispatcher_notes`. When you decide to invoke a subagent, read its full prompt at `.archon/subagents/<name>.md`.

**Invoke** via the generic wrapper (Bash tool, foreground):

```
python3 .claude/tools/archon-subagent.py \
  --name <subagent-name> \
  --slug <kebab-slug> \
  --directive-file <path-to-directive.md> \
  --write-domain '<glob>'        # repeat for multiple
```

The wrapper exits 0 on success, prints a one-line status, writes the report to `.archon/task_results/<name>-<slug>.md`, and the Archon CLI auto-archives a copy to `logs/iter-NNN/<name>-<slug>-report.md` for the dashboard. The dispatch semaphore caps concurrent subagent processes at `loop.max_parallel`; the wrapper resolves `--parent-slug` from `ARCHON_SUBAGENT_SLUG` so nested children inherit the hierarchy.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [frenzymath/Archon](https://github.com/frenzymath/Archon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
