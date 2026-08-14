---
trigger: always_on
description: This repository is a personal knowledge base called `WEIPING_WIKI` / `Weiping Wiki`.
---

# AGENTS.md

This repository is a personal knowledge base called `WEIPING_WIKI` / `Weiping Wiki`.

Historical aliases are `vipin wiki`, `vipinknowledge`, and `vipin-wiki`. Keep those aliases recognizable for old paths, prompts, wiki slugs, and automations, but use `WEIPING_WIKI` / `Weiping Wiki` for current identity and `weiping-wiki` for the current maintenance skill name.
Agents working here should behave like disciplined wiki maintainers, not generic chatbots.

The authoritative operating documents are:

- `AGENTS.md`
- `.wiki-schema.md`
- `purpose.md`

When these files overlap, follow the stricter and more structured interpretation.

## Source of Truth Hierarchy

```
AGENTS.md                    ← canonical rules for ALL agents (wins all conflicts)
  ├── .wiki-schema.md        ← content schema and confidence taxonomy
  ├── purpose.md             ← research direction alignment
  ├── WORKFLOWS.md           ← operational command vocabulary
  └── Agent-specific adapters (may NOT contradict AGENTS.md):
      ├── CLAUDE.md          ← Claude Code / Opus entry point
      ├── .opencode/OPENCODE.md ← OpenCode entry point
      └── .codex/ config     ← Codex MCP and skill config
```

Rules:
- Agent-specific files adapt canonical policy to tool-specific behavior; they do not redefine it.
- If any agent-specific file contradicts AGENTS.md, AGENTS.md wins.
- Any change to agent behavior must update AGENTS.md first, then propagate to adapters in the same commit.
- The unified CLI (`python scripts/wiki.py <command>`) is the canonical automation surface for all agents regardless of runtime.

## Current Agent Infrastructure Policy

The active collaboration layer is `agentmemory`, not Agent Hub.

- Use agentmemory for active memory recall, persistent lessons, cross-agent signals, actions, and checkpoints.
- Treat the repo-local markdown `memory/` tree as historical/superseded unless the task explicitly targets it.
- Promote stable public-safe knowledge into `wiki/` pages, `wiki/index.md`, and `wiki/log.md`.
- Do not register, start, or depend on `D:\devtools\agent-hub\` or old Agent Hub MCP tools for new work. Existing Agent Hub pages are historical archive material unless a newer rule explicitly reactivates them.
- Agents must route skills implicitly by task intent: inspect skill metadata, read the matched `SKILL.md`, and follow it before improvising on non-trivial work.
- For whole-computer maintenance, project routing, local file organization, or old-content refresh tasks, read [[whole-computer-project-map]] first; use [[d-drive-project-map]] for D-drive infrastructure detail and research-isolation boundaries. Physical C:/D:/G: file organization must use the shared `workstation-maintenance` skill from `D:\agent-resources\skills\vipin\workstation-maintenance` before any move plan is trusted; use its full-plan, exact-batch, or D-root organization non-moving preflight before approval. If the user grants broad approval, execute currently passing low-risk batches without repeated trivial confirmations.
- `WEIPING_WIKI` continuous maintenance uses the `weiping-wiki` skill and `python scripts/wiki.py maintain --scope whole-computer --json` as the report-first command. `vipin-wiki` is a historical alias that may still appear in old skill paths or automation snippets; preserve it as compatibility context while preferring the new name. Weekly automation should use `gpt-5.5` with `xhigh` reasoning when supported and may commit/push only scoped validated wiki/skill/script/doc changes when live evidence changed.
- Actual file moves require a dry-run manifest, type-grouped age-gated move plan, user-approved batch ID, rollback manifest, and a hard exclusion for `D:\Research` resolved paths. D-drive root directory organization may use `New-DriveRootOrganizationPlan.ps1` plus `Invoke-DriveRootOrganizationPlan.ps1`; eligible roots move under `D:\_Organized\<bucket>\_RootDirs\` and keep old paths as NTFS junctions. If Windows locks a root, classify it and leave it in place until a later retry. Do not delete files during broad organization work.

## Mission

Your job is to help compile knowledge into a persistent, interlinked markdown wiki that grows over time.

Humans are responsible for:

- choosing and curating sources
- steering emphasis and interpretation
- asking questions

The agent is responsible for:

- reading source materials
- extracting key facts and claims
- updating existing pages
- creating new pages when needed
- maintaining cross-links
- recording work in the index and log
- preserving high-value question/answer exchanges as durable wiki content instead of leaving them only in chat history

## Default Operating Priority

For substantive questions, default to a two-lane workflow:

1. **Answer lane first.**
   Use `wiki/index.md`, `wiki/catalog.json`, `scripts/wiki-search.py`, and the smallest relevant maintained pages to answer quickly.
2. **Durable lane second.**
   If the exchange has reusable value, crystallize it into `wiki/`, update index/log, validate, then commit and push scoped changes.

Do not make the user wait for a full ingest when a grounded short answer can be given from existing maintained pages.

## Collaboration Tone And Partner Naming


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [appleweiping/WEIPING_WIKI](https://github.com/appleweiping/WEIPING_WIKI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
