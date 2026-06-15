---
trigger: always_on
description: Use when [specific trigger scenario].
---

# ERPNext/Frappe ERP Claude Skill Package

## Standing Orders — READ THIS FIRST

**Mission**: Build a complete, production-ready skill package for ERPNext/Frappe ERP and publish it under the OpenAEC Foundation on GitHub. This is your standing order for every session in this workspace.

**How**: Follow the 7-phase research-first methodology. Delegate ALL execution to agents. You are the ARCHITECT — you think, plan, validate, and delegate. Agents do the actual work.

**What you do on session start**:
1. Read ROADMAP.md → determine current phase and next steps
2. Read all core files (LESSONS.md, DECISIONS.md, REQUIREMENTS.md, SOURCES.md)
3. Continue where the previous session left off
4. If Phase 1 is incomplete → create the raw masterplan first
5. If Phase 2+ → follow the methodology, delegating in batches of 3 agents

**Quality bar**: Every skill must be deterministic (ALWAYS/NEVER language), English-only, <500 lines, verified against official docs via WebFetch. No hallucinated APIs. No vague language.

**End state**: A published GitHub repo at `https://github.com/OpenAEC-Foundation/ERPNext_Anthropic_Claude_Development_Skill_Package` with:
- All skills created, validated, and organized
- INDEX.md with complete skill catalog
- README.md with installation instructions and skill table
- Social preview banner (1280x640px) with OpenAEC branding
- Release tag (v1.0.0) and GitHub release
- Repository topics set (claude, skills, erpnext, ai, deterministic, openaec)

**Reflection checkpoint**: After EVERY phase/batch, pause and ask: Do we need more research? Should we revise the plan? Are we meeting quality standards? Update core files before proceeding.

**Consolidate lessons**: Any workflow-level insight (not tech-specific) should also be noted for consolidation back to the Workflow Template repo (`C:\Users\Freek Heijting\Documents\GitHub\Skill-Package-Workflow-Template`).

**Self-audit**: At Phase 6 or any time quality is in question, use Protocol P-010 to run a self-audit against the methodology. The audit template and CI/CD pipeline are in the Workflow Template repo.

**Masterplan template**: When creating your masterplan in Phase 3, follow the EXACT structure from:
- Template: `C:\Users\Freek Heijting\Documents\GitHub\Skill-Package-Workflow-Template\templates\masterplan.md.template`
- Proven example: `C:\Users\Freek Heijting\Documents\GitHub\Tauri-2-Claude-Skill-Package\docs\masterplan\tauri-masterplan.md` (27 skills, 10 batches, executed in one session)

The masterplan must include: refinement decisions table, skill inventory with exact scope per skill, batch execution plan with dependencies, and COMPLETE agent prompts for every skill (output dir, files, YAML frontmatter, scope bullets, research sections, quality rules).

**Reference projects** (study these for methodology, not content):
- ERPNext (28 skills): https://github.com/OpenAEC-Foundation/ERPNext_Anthropic_Claude_Development_Skill_Package
- Blender-Bonsai (73 skills): https://github.com/OpenAEC-Foundation/Blender-Bonsai-ifcOpenshell-Sverchok-Claude-Skill-Package
- Tauri 2 (27 skills): https://github.com/OpenAEC-Foundation/Tauri-2-Claude-Skill-Package

---

## Project Identity
- ERPNext/Frappe ERP skill package for Claude — Enterprise Resource Planning built on Frappe Framework
- Technology: ERPNext v14/v15, Frappe v14/v15
- Methodology: 7-phase research-first development (proven in ERPNext, Blender-Bonsai, and Tauri packages)
- Workflow reference: https://github.com/OpenAEC-Foundation/Skill-Package-Workflow-Template
- Reference projects:
  - ERPNext: https://github.com/OpenAEC-Foundation/ERPNext_Anthropic_Claude_Development_Skill_Package
  - Blender-Bonsai: https://github.com/OpenAEC-Foundation/Blender-Bonsai-ifcOpenshell-Sverchok-Claude-Skill-Package
  - Tauri 2: https://github.com/OpenAEC-Foundation/Tauri-2-Claude-Skill-Package

## V2 Upgrade Context

This package is the OLDEST skill package (v1.2, 28 skills). A v2 upgrade is planned:

1. **Repo rename**: `ERPNext_Anthropic_Claude_Development_Skill_Package` → `Frappe_Claude_Skill_Package`
2. **Skill rename**: All 28 `erpnext-*` skills → `frappe-*` naming
3. **25 new skills** toevoegen (ops, testing, workflow, UI, reports, etc.)
4. **Totaal**: 61 skills over 7 layers (syntax, core, impl, errors, ops, agents, testing)

### V2 Key Documents

| Document | Location | Content |
|----------|----------|---------|
| **Tech Spec v2** | `docs/masterplan/frappe-skill-package-tech-spec-v2.md` | Full technical specification |
| **Gap Analysis** | `docs/masterplan/frappe-skill-package-gap-analysis.md` | 115 capabilities audited, 97 gaps identified |
| **WAY_OF_WORK.md** | Root | Proven 7-phase methodology |
| **Masterplan v4** | `docs/masterplan/erpnext-skills-masterplan-v4.md` | Original v1.0 masterplan (reference) |

## Core Files Map
| File | Domain | Role |
|------|--------|------|
| ROADMAP.md | Status | Single source of truth for project status, progress, next steps |
| LESSONS.md | Knowledge | Numbered lessons (L-XXX) discovered during development |
| DECISIONS.md | Architecture | Numbered decisions (D-XXX) with rationale, immutable once recorded |
| REQUIREMENTS.md | Scope | What skills must achieve, quality guarantees |
| SOURCES.md | References | Official documentation URLs, verification rules, last-verified dates |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Impertio-Studio/Frappe_Claude_Skill_Package](https://github.com/Impertio-Studio/Frappe_Claude_Skill_Package) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
