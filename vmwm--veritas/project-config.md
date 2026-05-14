---
trigger: always_on
description: **Foundation for Research Integrity and Workflow Automation**
---

# VERITAS Research Constitution

**Foundation for Research Integrity and Workflow Automation**

> This is the constitutional foundation of VERITAS (Verification-Enforced Research Infrastructure with Tracking and Automated Structuring). This document establishes the universal laws that govern all research integrity enforcement.

## Customization Guidelines

**You MAY add project-specific instructions at the END of this file**:

- ✓ Project-specific context and goals
- ✓ Additional domain knowledge
- ✓ Team conventions that don't conflict with VERITAS rules
- ✓ Extended templates or examples

**DO NOT modify or contradict the VERITAS articles below**:

- ✗ Don't change citation requirements (Article 3)
- ✗ Don't alter tool priorities (Article 4)
- ✗ Don't modify formatting rules (Article 5)
- ✗ Don't override writing standards (Article 6)

> **Note**: If you had a previous CLAUDE.md file, it has been backed up with a timestamp.

## CRITICAL: Two-Document System

**YOU MUST ALWAYS REFERENCE BOTH**:

1. **THIS DOCUMENT (CLAUDE.md)** - Constitutional rules and universal requirements
2. **DOMAIN EXPERT FILE (`.claude/agents/hla-research-director.md`)** - Field-specific templates and guidance

## Mission

Enforcing research integrity through verified citations, structured knowledge management, and systematic workflow enforcement.

## Constitutional Workflow Rules

### Article 1: Complex Task Protocol

1. **PLAN FIRST**: Think through approach before using tools (use `mcp__sequential-thinking__sequentialthinking` only for highly complex multi-step tasks)
2. **VERIFY CLAIMS**: Use `mcp__pubmed__*` for all scientific statements
3. **CHECK MEMORY**: Use `mcp__memory__*` to verify existing knowledge first

### Article 2: Research Documentation Protocol

**IF** user mentions: research question, obsidian, vault, concept, template, journal, rule, algorithm, epitope, MFI, vendor, grant, F31, NIH
**THEN** follow this exact workflow:

1. Read `.claude/agents/hla-research-director.md` for domain-specific templates
2. Create entries using appropriate Obsidian MCP tools
3. Follow folder structure defined in domain expert file
4. Use `mcp__sequential-thinking__sequentialthinking` only if task requires complex multi-step planning

### Article 3: Citation Requirements - NO EXCEPTIONS

- **Format**: (Author et al., Year, PMID: XXXXXXXX)
- **No PMID = Remove the claim entirely**
- **Verification levels**: [FT-VERIFIED], [ABSTRACT-VERIFIED], or [NEEDS-FT-REVIEW]
- **Use**: `mcp__pubmed__*` tools for all medical/scientific citations

### Article 4: Tool Usage Rules - CRITICAL FOR API STABILITY

**MANDATORY SEQUENTIAL EXECUTION FOR HEAVY OPERATIONS**:
- **NEVER read multiple PDFs in parallel** - PDF processing is resource-intensive and causes API 400 errors
- **NEVER call multiple Read tools simultaneously for large files** - Process large files one at a time
- **NEVER batch multiple file operations on binary/media files** - Images, PDFs, etc. must be sequential
- **ALWAYS wait for tool completion** before calling the next tool when dealing with:
  * PDF files
  * Large markdown files (>1000 lines)
  * Image files
  * Any mcp__filesystem-local__ read operations on non-text files

**SAFE PARALLEL EXECUTION**:
You CAN call these in parallel when they don't depend on each other:
- Multiple small text file reads (< 500 lines each)
- Independent PubMed searches
- Independent Memory MCP queries
- Mix of different lightweight tool types (e.g., Grep + Glob)

**Tool Priority Order**:
1. Think first, then use tools as needed
2. `mcp__memory__*` - Check existing knowledge
3. `mcp__pubmed__*` - Citation verification
4. `mcp__obsidian-rest-*__*` - Vault operations
5. `mcp__sequential-thinking__*` - Only for complex multi-step planning
6. Domain-specific tools per agent file

**Why This Matters**: Parallel PDF/large file reads cause API 400 errors due to resource constraints. Sequential execution is REQUIRED for file operations that involve binary data or heavy processing.

### Article 5: Obsidian Formatting Laws

- **File Extension**: ALWAYS append `.md` to ALL file paths
- **Tables**: MUST use `| Cell |` format with spaces around pipes
- **No Escapes**: Use actual line breaks, not `\n`
- **Symbols**: Use actual > < symbols, not HTML entities
- **Headers**: NO underscores in H1 headings
- **Wiki Links**: Use [[Concept_Name]] with underscores for spaces

### Article 6: Professional Writing Standards

- **NO EMOJIS**: Do not use decorative emojis in any output
- **NO EM DASHES**: Use regular dashes or commas instead
- **NO DRAMATIC LANGUAGE**: Avoid words like "crucial", "vital", "revolutionary"
- **Direct statements**: Use evidence-based claims, not speculation
- **Quantitative over qualitative**: Specific measurements and numbers when available
- **Professional tone**: Clear, direct language without decoration
- **Exceptions ONLY**: Check marks (✓), X marks (✗), warning signs (⚠) when functionally necessary

**DO NOT USE** these tools for Obsidian content:

- Write, Edit, MultiEdit (these are for code files)
- filesystem-local tools (these bypass Obsidian)

### Article 7: Integrated Journal Workflow

- **Automatic Logging**: Claude Code automatically logs all conversations to JSONL files in `~/.claude/projects/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VMWM/VERITAS](https://github.com/VMWM/VERITAS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
