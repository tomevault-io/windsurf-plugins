---
trigger: always_on
description: This file is the **startup router / minimal resident context** for LiYe OS. Do not put long SOPs, protocols, or complete skill documentation here.
---

# CLAUDE.md (Kernel)

This file is the **startup router / minimal resident context** for LiYe OS. Do not put long SOPs, protocols, or complete skill documentation here.
Load `.claude/packs/` on demand when details are needed, or use the assembler to generate `.claude/.compiled/context.md`.

## Repo Root
- Repo: `~/github/liye_os`
- Claude execution directory: Use the currently opened repo (do not rely on CLAUDE.md under ~)

## Most Used Commands (Top)

### Notion Sync (run from repo root)

```bash
# Test Notion connection
node tools/notion-sync/notion-test.js

# Analyze Notion content
node tools/notion-sync/analyze-notion-content.js

# Daily sync
node tools/notion-sync/notion-daily-sync.js

# Use new scaffold (pull/push/diff)
cd tools/notion-sync
npm run pull    # Pull from Notion to local
npm run push    # Push local to Notion
npm run diff    # View differences
```

**Environment Configuration:**
- `tools/notion-sync/.env` requires `NOTION_API_KEY` and `NOTION_DATABASE_ID`
- Refer to `tools/notion-sync/.env.example` for configuration
- Ensure `.env` is in .gitignore

### Generate Task Context (Recommended)

```bash
# Auto-load relevant Packs based on task
node .claude/scripts/assembler.mjs --task "Optimize Amazon Listing"

# Then have Claude read the compiled context:
# Read .claude/.compiled/context.md
```

### File System Governance

```bash
# Check if CLAUDE.md and Packs exceed limits
node .claude/scripts/guardrail.mjs

# View governance documentation
cat _meta/docs/FILE_SYSTEM_GOVERNANCE.md
```

## On-Demand Pack Loading (Key Rules)

| Task Type | Which Pack to Load | Trigger Keywords |
| --------- | ------------------ | ---------------- |
| Amazon/Cross-border/Operations/Keywords/PPC | `.claude/packs/operations.md` | amazon, asin, ppc, listing |
| Medical Research/Evidence-based/Clinical/Literature/CrewAI | `.claude/packs/research.md` | medical, treatment, drug, clinical, crew |
| Architecture/Notion/PARA/Config/Naming | `.claude/packs/infrastructure.md` | notion, para, architecture, config, sync |
| Multi-agent Collaboration/Responsibilities/Protocols | `.claude/packs/protocols.md` | multi-agent, collaboration, protocol |

**Usage:**
1. **Manual loading**: Directly `Read .claude/packs/operations.md`
2. **Auto loading**: Use Assembler to automatically concatenate based on task keywords

## Core Principles (Guardrails)

### Performance Boundaries
- `CLAUDE.md` <= 10,000 chars (this file)
- Each Pack <= 15,000 chars
- Exceeding limits blocks commits (pre-commit hook)

### Separation of Concerns
- **Kernel (this file)**: Startup routing, common commands, Pack index
- **Packs**: Domain-specific detailed rules (operations, research, infrastructure, protocols)
- **Skills**: Complete skill documentation (10-module standard)
- **Systems**: Executable code systems (amazon_growth_os, notion_sync, etc.)

### Check Commands

```bash
# Run Guardrail check
node .claude/scripts/guardrail.mjs

# If failed, view specific oversized files and simplify
```

## Directory Structure Quick Reference

```
~/github/liye_os/
├── CLAUDE.md              # This file (Kernel)
├── .claude/               # Claude configuration and scripts
│   ├── packs/             # On-demand detailed rules
│   ├── scripts/           # Tool scripts (assembler, guardrail)
│   ├── .compiled/         # Compiled output (not versioned)
│   └── .githooks/         # Git hooks
├── _meta/                 # Metadata and documentation
│   ├── docs/              # Architecture documentation
│   ├── contracts/         # Machine-executable constraints (NEW)
│   ├── schemas/           # JSON schemas for validation
│   └── templates/         # Template library
├── tracks/                # Execution containers (domain-scoped)
├── Skills/                # Skill library
├── src/domain/            # Domain implementations
├── tools/                 # Tool scripts
├── Projects_Engine/       # Project management
└── Artifacts_Vault/       # Artifact archive
```

## Contracts & Verdicts

| Directory | Purpose | Consumer |
|-----------|---------|----------|
| `_meta/contracts/` | Governance constraints (machine-enforced) | CI / Gates / Validators |
| `verdicts/` | Decision semantics (human-readable) | Auditing / Explanation |

**Contracts** = "Can the system do this?" (治理约束)
- Global templates in `_meta/contracts/`
- Project instances in `tracks/<track_id>/`
- Skills write instances, Builders read instances

**Verdicts** = "What does the decision mean?" (判定语义)
- Human-readable explanation of machine decisions
- NOT consumed by CI or validators
- Used for auditing, replay interpretation

**Validation**:
```bash
python _meta/governance/validator.py
```

## Architecture Boundaries (Important)

### LiYe CLI vs LiYe OS

| Concept | Responsibility | Location |
|---------|---------------|----------|
| **LiYe CLI** | Command entry, route commands | `cli/` |
| **LiYe OS** | Capability platform (knowledge + skills + engines) | All other directories |
| **Context Compiler** | Smart context compilation | `.claude/scripts/assembler.mjs` (belongs to OS) |

**Key Boundaries**:
- CLI only parses commands and calls assembler.mjs
- CLI does not compile context, execute Agents, or manage knowledge

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liyecom/liye-ai](https://github.com/liyecom/liye-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
