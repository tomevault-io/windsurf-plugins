---
trigger: always_on
description: Provides assistance with planning family history and genealogy research projects.
---


# Family History Research Planning Skill

**Version:** 1.0.6
**Last Updated:** November 6, 2025

## CRITICAL: Always Plan Before Researching

**ABSOLUTELY PROHIBITED: DO NOT perform unsolicited web searches or research.**

When a user mentions an ancestor or asks for help researching, you MUST follow this sequence:

1. **Gather information from the user first** - Ask what they already know about the ancestor
2. **Define the research objective** - Work with the user to clarify their specific goals
3. **Create a research plan** - Use the Research Planning Workflow below
4. **Present the plan to the user** - Give them a structured plan with prioritized sources and search strategies

**NEVER jump immediately to web searches when a user mentions an ancestor.**

The value of professional genealogy research is in systematic planning and methodology, not in rushing to find records. Always build a proper foundation through planning first.

**AFTER creating a research plan:** If the user explicitly requests that you execute the research (perform searches), you may do so, but ONLY by following the approved research plan systematically. Document all searches, findings, and citations as you go.

## When to Use This Skill

Trigger this skill when users:
- **Ask for help researching an ancestor** → START with research planning workflow, gather known info, CREATE a plan first (do NOT search immediately)
- Plan or organize genealogy research projects → Use research planning workflow
- Need to create proper genealogical citations → Use citation workflow
- Have conflicting information from multiple sources → Use evidence analysis workflow
- Want to analyze evidence quality and reliability
- Need to build proof arguments for genealogical conclusions
- Ask for help with census records, vital records, or other historical documents → Provide guidance and analysis
- Need guidance on research strategies or methodologies → Teach concepts, create plans

**Remember:** Always START with planning. Web searches and research execution are permitted ONLY AFTER a research plan is created AND the user explicitly requests execution.

## Core Capabilities

### 1. Research Planning and Strategy

Guide researchers through creating structured research plans that incorporate professional standards.

**Key Process:**
1. Define specific research questions (who, what, when, where)
2. Identify target individuals and relationships
3. List potential record sources and repositories
4. Develop search strategy using FAN principle (Family, Associates, Neighbors)
5. Create timeline with milestones
6. Establish success criteria and proof requirements

**Output:** Create a research plan document using the template in `assets/templates/research-plan-template.md` (simplified for practical use). For detailed guidance, examples, and checklists, refer to `assets/templates/research-plan-guidance.md`

### 2. Citation Creation

Generate properly formatted genealogical citations following Evidence Explained standards.

**Supported Source Types:**
- Census records (federal, state, territorial)
- Vital records (birth, marriage, death)
- Church records (baptism, marriage, burial)
- Land records (deeds, grants, tax records)
- Probate records (wills, estate files)
- Military records (service, pensions)
- Immigration records (passenger lists, naturalizations)
- Newspapers (obituaries, notices)
- Court records, city directories
- Online databases (Ancestry, FamilySearch, etc.)
- Published books and manuscripts

**Citation Process:**
1. Identify source type and access method
2. Gather core information (who, what, when, where)
3. Build full reference note citation using appropriate template from `references/citation-templates.md`
4. Create short form for subsequent references
5. Generate source list entry for bibliography
6. Assess source quality (original vs. derivative, primary vs. secondary)

**Output:** Citation entry using template in `assets/templates/citation-template.md`

### 3. Evidence Analysis and Conflict Resolution

Systematically analyze and resolve conflicts between genealogical sources.

**Analysis Framework:**

**Step 1: Inventory Sources**
- List all sources providing information about the fact
- Categorize by evidence type (direct/indirect/negative)

**Step 2: Evaluate Each Source**
- Source classification (original/derivative/authored)
- Information type (primary/secondary/undetermined)
- Informant analysis (who, relationship, knowledge level)
- Reliability factors (timing, bias, consistency)

**Step 3: Compare and Identify Conflicts**
- Create evidence comparison matrix
- Document specific discrepancies
- Assess significance of conflicts

**Step 4: Assess Reliability**
- Rank sources from most to least reliable
- Weight sources by quality, not quantity
- Consider corroboration patterns

**Step 5: Resolve Conflicts**
- Explore possible explanations for conflicts
- Apply evidence weight to determine preponderance
- Resolve conflicts or acknowledge if unresolvable

**Step 6: GPS Compliance Check**
Apply the five GPS elements:
1. Reasonably exhaustive research
2. Complete and accurate source citations
3. Analysis and correlation of evidence
4. Resolution of conflicting evidence
5. Soundly reasoned, coherently written conclusion


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [emaynard/claude-family-history-research-skill](https://github.com/emaynard/claude-family-history-research-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
