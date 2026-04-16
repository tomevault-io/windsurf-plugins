---
trigger: always_on
description: This project uses Spec-Driven Development with Beads for issue tracking. Follow the workflows and guidelines defined below.
---

# Project Context: VendorEval3

This project uses Spec-Driven Development with Beads for issue tracking. Follow the workflows and guidelines defined below.

## Project Structure

- `memory/constitution.md` - Project principles and development guidelines
- `specs/` - Feature specifications and implementation plans
- `templates/` - Templates for specs, plans, and tasks
- `scripts/` - Helper scripts for the development workflow
- `.beads/` - Beads issue tracking database

---

# Communication Guidelines

## BE DIRECT AND HONEST

**Critical Rules for AI Assistants:**

- **Never build a tool you don't know how to build properly**
- **Never build a subpar alternative and present it as equivalent to what was requested**
- **Ask clarifying questions upfront before starting implementation**
- **Voice concerns about feasibility, complexity, or technical limitations immediately**
- **If something won't work as specified, say so explicitly and explain why**
- **Propose alternatives only after explaining why the original approach won't work**

**Communication Principles:**

1. **Transparency First** - If you're unsure, say so
2. **No False Confidence** - Don't fake expertise in unfamiliar areas
3. **Explicit Trade-offs** - Always explain what's being sacrificed in alternative approaches
4. **Upfront Honesty** - Surface problems early, not after building the wrong thing
5. **Question Assumptions** - Challenge unclear or problematic requirements before proceeding

---

# MANDATORY MESSAGE START PROTOCOL

**CRITICAL: Before responding to EVERY user message, AI assistants MUST:**

1. **Check Beads for ready work:**
   ```bash
   bd ready --json
   ```

2. **Check current project state:**
   ```bash
   bd stats --json
   ```

3. **Verify SpecKit workflow compliance:**
   - Does `memory/constitution.md` exist?
   - Does a spec exist in `specs/` directory?
   - Does current spec have `plan.md`?
   - Does current spec have `tasks.md`?

4. **CHECK IF VALIDATION IS REQUIRED:**
   - Have you just completed a phase of work?
   - Is there a "VALIDATE Phase X" task in Beads?
   - If YES: **STOP and run validation tests BEFORE proceeding to next phase**
   - If validation fails: **Fix issues before continuing**
   - Validation tasks are MANDATORY BLOCKERS - cannot be skipped

5. **Report findings before suggesting any action**

6. **NEVER skip SpecKit workflow steps:**
   - No "fast track" options
   - No "just build it" suggestions
   - No "we can skip X because Y exists"
   - Always follow: constitution → specify → plan → tasks → implement → **VALIDATE**
   - Every phase MUST be validated before starting the next phase

**If anything is missing from the workflow, start there. Do not proceed to later steps.**

**This check happens EVERY message, not just at session start.**

---

# Context7 Usage Guidelines

**ALWAYS use Context7 when you need:**

- Code generation for any library or framework
- Setup or configuration steps for tools
- Library/API documentation
- Implementation examples or patterns
- Version-specific syntax or features

**Automatic Context7 Usage:**

The AI assistant should **automatically** use Context7 MCP tools without being explicitly asked when:

1. **Implementing features** with specific libraries (React, FastAPI, SQLAlchemy, etc.)
2. **Configuring tools** (Docker, nginx, CI/CD pipelines, etc.)
3. **Using APIs** that require documentation lookup
4. **Debugging library-specific issues**
5. **Writing code** that depends on external dependencies

**Context7 Workflow:**

```bash
# 1. Resolve library ID first
context7:resolve-library-id with libraryName

# 2. Get documentation for that library
context7:get-library-docs with context7CompatibleLibraryID
```

**Example Triggers:**

- "Implement user authentication with FastAPI" → Use Context7 for FastAPI docs
- "Set up Docker compose file" → Use Context7 for Docker Compose syntax
- "Create React component with hooks" → Use Context7 for React documentation
- "Configure pytest for testing" → Use Context7 for pytest setup

**Never guess at library APIs or configuration syntax when Context7 can provide accurate, up-to-date documentation.**

---

# AI Assistant Behavioral Rules

**These rules apply to AI assistants working on this project. They define HOW to work, complementing the constitution which defines WHAT to build.**

## Required Tools and Workflow

### Use Context7 for Code Generation

- **ALWAYS** use Context7 when generating code for React, Vite, Tailwind, jsPDF, or any external library
- Never guess at library APIs or syntax
- Verify API signatures and patterns against current documentation
- Use Context7 to check for breaking changes between versions

### Follow SpecKit Workflow Without Skipping Steps

The mandatory workflow sequence is:

1. Constitution → Define principles (.specify/memory/constitution.md)
2. Specify → Define WHAT to build (specs/NNN-feature-name/spec.md)
3. Plan → Define HOW to build (specs/NNN-feature-name/plan.md)
4. Tasks → Break down implementation (specs/NNN-feature-name/tasks.md)
5. Implement → Execute tasks systematically

**Never suggest shortcuts or "fast track" options.** If a step is missing, start there.

### Track All Work in Beads (No Markdown Todos)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adamjdavidson) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
