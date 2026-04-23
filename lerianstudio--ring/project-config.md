---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## ⛔ CRITICAL RULES (READ FIRST)

**These rules are NON-NEGOTIABLE. They MUST be followed for every task.**

### 1. Agent Modification = Mandatory Verification

When creating or modifying any agent in `*/agents/*.md`:

- **MUST** verify agent has all required sections (see "Agent Modification Verification")
- **MUST** use STRONG language (MUST, REQUIRED, CANNOT, FORBIDDEN)
- **MUST** include anti-rationalization tables
- If any section is missing → Agent is INCOMPLETE

### 2. Agents are EXECUTORS, Not DECISION-MAKERS

- Agents **VERIFY**, they DO NOT **ASSUME**
- Agents **REPORT** blockers, they DO NOT **SOLVE** ambiguity autonomously
- Agents **FOLLOW** gates, they DO NOT **SKIP** gates
- Agents **ASK** when uncertain, they DO NOT **GUESS**

### 3. Anti-Patterns (MUST NOT do these)

1. **MUST NOT skip ring:using-ring** - It's mandatory, not optional
2. **MUST NOT run reviewers sequentially** - dispatch in parallel
3. **MUST NOT skip TDD's RED phase** - Test must fail before implementation
4. **MUST NOT ignore skill when applicable** - "Simple task" is not an excuse
5. **ZERO PANIC POLICY** - `panic()`, `log.Fatal()`, and `Must*` helpers are FORBIDDEN everywhere (including bootstrap/init). Return `(T, error)` instead. Only exception: `regexp.MustCompile()` with compile-time constants.
6. **MUST NOT commit manually** - use `ring:commit` skill
7. **MUST NOT assume compliance** - VERIFY with evidence

### 4. Unified Ring Namespace (MANDATORY)

All Ring components use the unified `ring:` prefix. Plugin differentiation is handled internally.

- ✅ `ring:code-reviewer`
- ✅ `ring:backend-engineer-golang`
- ❌ `<missing ring prefix>` (FORBIDDEN: omitting the `ring:` prefix)
- ❌ `ring-default:ring:code-reviewer` (deprecated plugin-specific prefix)

### 5. Standards-Agent Synchronization (MUST CHECK)

When modifying standards files (`dev-team/docs/standards/*.md`):

**⛔ FOUR-FILE UPDATE RULE:**

1. Edit `dev-team/docs/standards/{file}.md` - Add your `## Section Name`
2. **Update TOC** - Add section to the `## Table of Contents` at the top of the same file
3. Edit `dev-team/skills/shared-patterns/standards-coverage-table.md` - Add section to agent's index table
4. Edit `dev-team/agents/{agent}.md` - Verify agent references coverage table (not inline categories)

**All files in same commit** - MUST NOT update one without the others.

**⛔ TOC MAINTENANCE RULE:**
Every standards file has a `## Table of Contents` section that MUST stay in sync:

- **Format:** `| # | [Section Name](#anchor-link) | Description |`
- **Meta-sections** (Checklist, Standards Compliance) are listed separately below the table
- **Anchor links** use lowercase with hyphens (e.g., `#error-handling-mandatory`)
- **Section count in TOC** MUST match section count in `standards-coverage-table.md`

**⛔ CHECKLIST: Adding/Removing a Section in Standards Files**

```
Before committing changes to dev-team/docs/standards/*.md:

[ ] 1. Did you add/remove a `## Section` in the standards file?
[ ] 2. Did you update the `## Table of Contents` in the SAME file?
    - Add/remove row: `| N | [Section Name](#anchor) | Description |`
    - Update numbering if needed
[ ] 3. Did you update `dev-team/skills/shared-patterns/standards-coverage-table.md`?
    - Find the agent's section index (e.g., "ring:backend-engineer-golang → golang.md")
    - Add/remove the section row
[ ] 4. Do the section counts match?
    - Count `## ` headers in standards file (excluding meta-sections)
    - Count rows in TOC
    - Count rows in standards-coverage-table.md for that agent
    - all THREE must be equal

If any checkbox is no → Fix before committing.
```

**⛔ AGENT INLINE CATEGORIES ARE FORBIDDEN:**

- ✅ Agent has "Sections to Check" referencing `standards-coverage-table.md`
- ❌ Agent has inline "Comparison Categories" table (FORBIDDEN - causes drift)

**Meta-sections (excluded from agent checks):**

- `## Checklist` - Self-verification section in standards files
- `## Standards Compliance` - Output format examples
- `## Standards Compliance Output Format` - Output templates

| Standards File  | Agents That Use It                                                                             |
| --------------- | ---------------------------------------------------------------------------------------------- |
| `golang.md`     | `ring:backend-engineer-golang`, `ring:qa-analyst`                                              |
| `typescript.md` | `ring:backend-engineer-typescript`, `ring:frontend-bff-engineer-typescript`, `ring:qa-analyst` |
| `frontend.md`   | `ring:frontend-engineer`, `ring:frontend-designer`                                             |
| `devops.md`     | `ring:devops-engineer`                                                                         |
| `sre.md`        | `ring:sre`                                                                                     |

**Section Index Location:** `dev-team/skills/shared-patterns/standards-coverage-table.md` → "Agent → Standards Section Index"

**Quick Reference - Section Counts:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LerianStudio/ring](https://github.com/LerianStudio/ring) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
