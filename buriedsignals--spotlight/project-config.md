---
trigger: always_on
description: Runtime contract for the Spotlight OSINT investigation system
---


# Spotlight — Runtime Contract

This file defines the portable contract for the Spotlight investigation system. Any runtime that can (a) read this file + the skills in `skills/`, (b) dispatch the 13 verbs to native tools, and (c) spawn sub-agents can run Spotlight investigations.

Target runtimes: **pi** (https://pi.dev — native AGENTS.md + SKILL.md support), **Hermes** (production Mycroft agent), **Goose** (served as an extension pack), **Codex CLI**, **Gemini CLI**, and any harness driving a local OpenAI-compatible endpoint (llama-server, Ollama, Exoscale, vLLM). Claude Code users stay on the existing marketplace plugin at `~/buried_signals/tools/skills/spotlight/` — this repo exists for non-Claude runtimes.

Per-runtime wiring is documented in `docs/integrations.md`.

## Tool Verb Registry

Fixed vocabulary of abstract operations. Every runtime adapter MUST implement all verbs. If a verb is unsupported, the adapter raises an explicit error at load time.

| Verb | Signature | Semantics | Universal backing |
|------|-----------|-----------|-------------------|
| `fetch` | `fetch(url, output_path)` | Scrape URL content, save to file | `firecrawl scrape` |
| `search` | `search(query, output_path, limit)` | Web search, save results to file | `firecrawl search` |
| `read-file` | `read-file(path)` | Read file contents | filesystem |
| `write-file` | `write-file(path, content)` | Write file (full overwrite) | filesystem |
| `edit-file` | `edit-file(path, old, new)` | Targeted string replacement | filesystem |
| `list-files` | `list-files(pattern)` | Glob/search for files matching pattern | glob |
| `grep-files` | `grep-files(pattern, path)` | Search file contents by regex | ripgrep |
| `execute-shell` | `execute-shell(command)` | Run shell command, return stdout + stderr | shell subprocess |
| `spawn-agent` | `spawn-agent(agent_id, prompt, config)` | Launch sub-agent with prompt and config | runtime-specific |
| `wait-agent` | `wait-agent(handle)` | Block until agent completes, return output | runtime-specific |
| `invoke-skill` | `invoke-skill(skill_id)` | Load skill instructions into current context | runtime-specific |
| `query-vault` | `query-vault(vault_path, query)` | Search knowledge vault for context | `BUN_INSTALL="" qmd query` |
| `vault-write` | `vault-write(vault_path, note_path, content)` | Write note to vault and update registry | `obsidian` CLI |

## Agent Manifests

### investigator

```yaml
name: investigator
description: Plans and executes OSINT investigations using structured methodology
iteration_limit: 80
allowed_verbs:
  - fetch
  - search
  - read-file
  - write-file
  - edit-file
  - list-files
  - grep-files
  - invoke-skill
  - query-vault
  - execute-shell
disallowed_verbs:
  - spawn-agent
preferred_model:
  claude: opus
  gemini: gemini-2.5-pro
  gpt: gpt-4o
  local: HauhauCS/Qwen3.6-27B-Uncensored-HauhauCS-Aggressive
  fallback_note: Investigation quality degrades significantly on lighter models. Local ship — Qwen3.6 27B Uncensored (dense, thinking mode, zero refusals, multimodal) for 24+ GB Macs; Gemma 4 26B A4B MoE for 16 GB Macs. Both unsloth GGUF. Q4_K_M recommended. Native vision for scanned docs + satellite imagery.
vault_context:
  enabled: true
  query_on_load: true
```

The investigator operates in two modes:

- **PLANNING** — Analyzes the brief, queries the vault for prior work, designs methodology, writes `cases/{project}/data/methodology.json`
- **EXECUTION** — Follows approved methodology, executes research using `fetch` and `search`, writes `cases/{project}/data/findings.json` and appends to `cases/{project}/data/investigation-log.json`

Full prompt bundle: `agents/investigator.md`.

### fact-checker

```yaml
name: fact-checker
description: Independent verification of investigation findings using SIFT methodology
iteration_limit: 50
allowed_verbs:
  - fetch
  - search
  - read-file
  - write-file
  - list-files
  - grep-files
  - invoke-skill
  - query-vault
  - execute-shell
disallowed_verbs:
  - spawn-agent
preferred_model:
  claude: opus
  gemini: gemini-2.5-pro
  gpt: gpt-4o
  local: gemma-4-26B-A4B-it
  fallback_note: Fact-checking accuracy degrades significantly on lighter models. Local ship — unsloth/gemma-4-26B-A4B-it-GGUF (Q4_K_M for 24GB+ Macs, Q6_K_XL for 48GB+). Native vision for scanned docs + satellite imagery.
vault_context:
  enabled: true
  query_on_load: true
```

The fact-checker operates independently from the investigator. Spawned with its own context, it reads only the investigator's JSON output — not their reasoning — and writes `cases/{project}/data/fact-check.json` with per-claim verdicts and evidence trails.

**Verdict taxonomy:** `verified` | `unverified` | `disputed` | `false`

Full prompt bundle: `agents/fact-checker.md`.

## Skill Registry

Skills are markdown playbooks loaded via `invoke-skill(skill_id)`. Each skill lives in `skills/{skill_id}/SKILL.md` with optional reference files in `skills/{skill_id}/references/`.

| Skill ID | Path | Description | Invocable By |
|----------|------|-------------|--------------|
| `spotlight` | `skills/spotlight/SKILL.md` | Investigation orchestrator — pipeline phases, gates, cycle evaluation | orchestrator (top-level) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [buriedsignals/spotlight](https://github.com/buriedsignals/spotlight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
