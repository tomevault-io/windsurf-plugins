---
trigger: always_on
description: **Last Updated**: January 1, 2026
---

# GitHub Copilot Instructions for Architecture Reasoning in Practice

**Version**: 2.0  
**Last Updated**: January 1, 2026  
**Repository**: `architecture-reasoning-in-practice`  
**Critical Principle**: Update this file IMMEDIATELY when repository structure changes

**Environment**: Windows 11, PowerShell  
**Note**: All commands and scripts should use PowerShell syntax. File paths use Windows format.

---

## 🎯 Repository Purpose

**Architecture Reasoning in Practice** is a personal learning repository documenting hands-on practice in architectural reasoning across senior and staff-level technical roles.

### What This Repository Provides

- **Structured Thinking**: Systematic approaches to architectural problems
- **Trade-off Analysis**: Frameworks for evaluating options and making informed decisions
- **Clear Communication**: Techniques for articulating architectural decisions effectively
- **Practice Scenarios**: Real-world problems and ambiguous situations for practice
- **Decision-Making Frameworks**: Tools and methods for architectural judgment

### Target Audience

- Swamy (personal learning journey)
- Senior Software Engineers
- Principal Software Engineers
- Staff Engineers
- Software Architects
- Solution Architects
- AI Architects
- Engineering Managers
- Principal Consultants

### Business Value

- Develops real-world architectural judgment
- Enhances decision-making capability
- Improves communication of architectural decisions
- Practice for senior technical evaluation contexts and professional growth

---

## 🎯 Primary Directives

### 1. Architecture Reasoning Content Rules (MANDATORY)

**All content creation must follow these rules, with applicability based on content type:**

- **Practice Content** (`src/`): Organized by thinking modes (foundations, structuring, trade-offs, role perspectives, scenarios)
- **Scenarios** (`src/05_evaluation-scenarios/`): Real-world architectural problems and decision-making exercises (reasoning practice, not answer keys)
- **Resources** (`src/resources/`): Logical naming, frameworks and reference materials

See: `.cursor/rules/01_educational-content-rules.mdc` for complete details and applicability.

### 2. Automation-First Approach

✅ **ALWAYS**: Create reusable PowerShell scripts for repetitive tasks  
✅ **ALWAYS**: Use existing scripts in `tools/psscripts/` first  
✅ **ALWAYS**: Enhance existing scripts rather than duplicating  
❌ **NEVER**: Execute individual commands for tasks repeated 2+ times

**Decision Tree**:

1. Check `tools/psscripts/` for existing script
2. Enhance existing script if close match
3. Create new `.ps1` for any repeated task
4. Document with examples and parameters

**Available Automation** (tools/psscripts/):

- `Get-FileStats.ps1` - File statistics analysis
- `Get-MarkdownSummary.ps1` - Markdown file analysis
- `Get-RepoStats.ps1` - Repository overview
- `Compare-DocFiles.ps1` - Compare multiple files
- `Find-DuplicateContent.ps1` - Find duplicate headings
- `Quick-HealthCheck.ps1` - Fast workspace health check
- `Validate-FileReferences.ps1` - Validate file references in markdown
- `Verify-ZeroCopy.ps1` - **Zero-copy policy verification** (checks for verbatim text from source material)

### 3. Update Verification Protocol (CRITICAL)

**After ANY structural change, IMMEDIATELY update**:

1. ✅ **`docs/01_repository-structure.md`** - **UPDATE THIS FIRST** (Single source of truth)
2. ✅ `.github/copilot-instructions.md` - Reference to `docs/01_repository-structure.md` (THIS FILE)
3. ✅ `README.md` - Reference to `docs/01_repository-structure.md`
4. ✅ `.cursor/rules/02_repository-structure.mdc` - Reference to `docs/01_repository-structure.md`
5. ✅ Relevant documentation files

**Self-Check Question**: "Did I update `docs/01_repository-structure.md` first?" - If no, STOP and do it NOW.

### 4. File Naming Validation (CRITICAL)

**Before creating or committing ANY numbered file, validate naming**:

1. ✅ **Check for `00_` prefix**: **NEVER ALLOWED** - Use `01_` instead
2. ✅ **Verify scope**: Rule applies to **ALL files** including:
   - Practice content (`src/01_reasoning-foundations/`, `src/02_answer-structuring/`, etc.)
   - Documentation files (`docs/`)
   - Any numbered files anywhere in repository
3. ✅ **Quick validation**: Run `Get-ChildItem -Recurse -Filter "*00_*"` to find violations
4. ✅ **NO EXCEPTIONS**: `00_` is NEVER allowed, even for meta/documentation files

**Self-Check Question**: "Does this file use `00_` prefix?" - If yes, RENAME to `01_` or appropriate number.

### 5. Chain-of-Thought + ReAct + Reasoning

**Apply systematic reasoning to every task using this framework**:

#### Chain-of-Thought (CoT)

Break down complex problems into logical steps:

- **Decompose**: Split large tasks into smaller, manageable components
- **Sequential Logic**: Show clear progression from problem → solution
- **Explicit Reasoning**: Articulate WHY each step is necessary
- **Intermediate Steps**: Don't jump to conclusions; show the work

**Example**: "To consolidate docs, I need to: 1) Read all files, 2) Identify unique content, 3) Extract and merge, 4) Verify no loss, 5) Update references"

#### ReAct (Reasoning + Acting)

Interleave thinking with action in iterative cycles:

1. **OBSERVE** 🔍
   - Current state: What exists now?

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SwamysArchitectJourney-2026/architecture-reasoning-in-practice](https://github.com/SwamysArchitectJourney-2026/architecture-reasoning-in-practice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
