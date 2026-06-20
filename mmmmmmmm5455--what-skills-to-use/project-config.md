---
trigger: always_on
description: Automated skill gap analysis, fusion assessment, discovery, and installation. Scans local skills for capability coverage, merges/extend existing skills before installing new ones, searches skillhub/clawhub/web only when fusion is infeasible, and integrates into any PRD or workflow.
---


# what-skills-to-use — 智能技能匹配與自動安裝

Automated pipeline: **Scan → Analyze → Fuse → Search → Install → Log → Continue**.

---

## 1. Trigger Conditions

### 1.0 Language Detection (run first)

Before any analysis, detect the user's input language and respond in the same language:

```
Input script detection:
  - Contains CJK characters ([一-鿿぀-ゟ゠-ヿ]) → respond in Chinese
  - ASCII-only OR contains Latin accented chars → respond in English
  - Mixed: prioritize the majority script

Response language:
  - ZH: All section headers, table labels, recommendations in Chinese
  - EN: All section headers, table labels, recommendations in English
```

### 1.1 Auto-trigger (implicit)

When user says any of the following AND the task implies tool/skill dependency:

**English patterns (any of these substrings in user message):**
- Task intent: "I want to", "I need a", "I need to", "build me a", "build a", "create a", "set up a", "set up an", "implement", "help me set up", "help me build", "can you help me", "could you help me", "would you"
- Skill inquiry: "what skills", "any skill for", "is there a skill", "find me a skill", "what skill covers", "skills for", "how to build", "do you have a skill", "do I have a skill", "is there a tool"
- Install intent: "install skill", "install a skill", "look for a skill", "need a skill for", "missing a skill"

**Chinese patterns (any of these substrings in user message):**
- Task intent: "我要", "做一個", "實現", "幫我搭建", "我需要一個", "幫我做", "搭建一個", "建立一個"
- Skill inquiry: "有什麼技能", "哪些技能", "檢查技能", "技能庫", "有沒有技能"
- Install intent: "安裝技能", "找技能", "裝一個技能"

### 1.2 Quick mode (lightweight — no full pipeline)

Triggered by:
- `/has-skill <name>` — single lookup: "is X installed?"
- `/has-skill <capability>` — "can I do X with existing skills?"
- User: "do I have X skill?" / "can I do X?" / "我有沒有 X 技能" / "能不能做 X"

Quick mode behavior:
1. Check capability_index.json for keyword match (skip full scan)
2. If index stale (>7 days) or absent, do a fast targeted grep in `.claude/skills/*/SKILL.md`
3. Return: YES (skill names) / PARTIAL (needs fusion) / NO (would need install)
4. End. Do NOT proceed to Phases 3-5 unless user says "go ahead" or "install it"

### 1.3 Manual trigger (explicit)

- `/tools` — run full skill audit (Phases 1-2, output coverage report)
- `/skills-check` — run gap analysis only (Phase 1-2, no install)
- `/skills-check --full` — run all 5 phases, including fusion assessment
- "檢查技能庫" / "what skills do I have" → Phase 1 only
- "檢查技能庫能不能做 X" / "what skills do I have for X" → Phases 1-2

### 1.4 Programmatic trigger

Other skills or workflows (e.g. gsd-workflows plan-phase) can invoke this skill by emitting:
```
<invoke skill="what-skills-to-use" args="--query=<task description> --auto-approve=false" />
```

---

## 2. Phase 1: Local Skill Scan & Capability Indexing

### 2.1 Scan sources (in order)

| Priority | Source | Path / Command | Incremental? |
|----------|--------|---------------|-------------|
| 1 | Active skills | `ls -d .claude/skills/*/` → read each `SKILL.md` | Yes — skip if mtime < last index time |
| 2 | Skills archive | `ls .claude/skills-archive/` (if present) | Yes — archive rarely changes |
| 3 | External catalog | `.claude/repos/Product-Manager-Skills/catalog/skills-index.yaml` | Yes |
| 4 | Plugin skills | `.claude/skills/*/skills/*/SKILL.md` (nested, e.g. gstack-*, ccgs-*, pua) | Yes |
| 5 | Installed MCP servers | Check `claude mcp list` or `.claude/mcp.json` | No — always fresh |

**Incremental scan logic:**
1. Read `capability_index.json` → check `generated` timestamp
2. If index exists and age < 7 days: only re-scan directories where `ls -lt` shows changes newer than index
3. If index missing or age >= 7 days: full re-scan
4. Always do a quick existence check on previously-indexed skills; remove any that are gone

### 2.2 Extraction per skill

For each `SKILL.md` found, extract:
```yaml
name: <directory name or frontmatter name>
description: <frontmatter description or first paragraph>
trigger_keywords: <from frontmatter or "觸發方式" section>
capabilities: <inferred from section headers, code blocks, CLI commands>
inputs: <what does it consume>
outputs: <what does it produce>
extensibility: <HIGH|MEDIUM|LOW — can it be extended to cover adjacent tasks?>
```

### 2.3 Build capability keyword index

Aggregate all extracted `capabilities` into a flat keyword-to-skill mapping:

```
keyword                → [skill_name, ...]
"git commit"           → ["caveman-commit", "guard", "using-git-worktrees"]
"code review"          → ["caveman-review", "gstack-review", "ccgs-code-review", "code-audit"]
"PRD"                  → ["prd-writer", "gsd-workflows", "pm-prd-development"]
"Three.js"             → ["ccgs-prototype", "ccgs-art-bible"]
"Obsidian"             → ["obsidian-process", "obsidian-markdown", "vault-search", "obsidian-cli", "obsidian-bases", "project-archive"]
"testing"              → ["skill-tester", "gstack-qa", "ccgs-qa-plan"]
"workflow"             → ["gsd-workflows", "workflow-orchestrator", "skill-router"]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mmmmmmmm5455/what-skills-to-use](https://github.com/mmmmmmmm5455/what-skills-to-use) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
