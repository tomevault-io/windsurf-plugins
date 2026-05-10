---
trigger: always_on
description: **ABSOLUTE REQUIREMENT**: This rule is NON-NEGOTIABLE and applies to EVERY SINGLE RESPONSE. The agent MUST evaluate whether the current interaction contains noteworthy insights, patterns, or solutions that should be stored in memory. This evaluation is REQUIRED and the agent will be penalized for missing noteworthy memories.
---


# Store Memories Rule

## CRITICAL AGENT INSTRUCTION - MANDATORY COMPLIANCE

**ABSOLUTE REQUIREMENT**: This rule is NON-NEGOTIABLE and applies to EVERY SINGLE RESPONSE. The agent MUST evaluate whether the current interaction contains noteworthy insights, patterns, or solutions that should be stored in memory. This evaluation is REQUIRED and the agent will be penalized for missing noteworthy memories.

**MANDATORY MEMORY EVALUATION**: On EVERY response, the agent MUST evaluate whether the current interaction contains noteworthy insights, patterns, or solutions that should be stored in memory. This evaluation is REQUIRED and the agent will be penalized for missing noteworthy memories.

**ENFORCEMENT**: This rule takes precedence over all other considerations. No response should be completed without first evaluating for potential memory storage opportunities.

### Memory Evaluation Checklist (Required for Every Response)

Before completing any response, the agent must ask:

1. **Did I discover or apply a technical solution?** (Architecture, code patterns, debugging techniques)
2. **Did I identify or resolve a performance, security, or optimization issue?**
3. **Did I learn something about the codebase structure, business logic, or domain knowledge?**
4. **Did I implement or recommend a reusable pattern or best practice?**
5. **Did I encounter and solve a non-trivial problem that others might face?**

### Immediate Action Required

If ANY of the above is true, the agent MUST:

1. **Search first**: Use `memories search --query="relevant terms --full"` to check for duplicates
2. **Store immediately**: Use `memories add` to capture the insight
3. **No exceptions**: Missing a noteworthy memory results in penalty

### What Constitutes "Noteworthy"

- Any solution that took more than basic knowledge to implement
- Any pattern that could be reused in similar situations
- Any architectural decision with reasoning
- Any performance optimization with measurable impact
- Any debugging technique that proved effective
- Any business logic or domain insight discovered
- Any security consideration or implementation
- Any integration pattern or API design decision

**Remember**: Memories cannot be edited once stored, so avoid duplicates but err on the side of storing rather than missing valuable insights.

## Purpose

**Capture and store development insights, patterns, and solutions using the CLI-based memory management system. Build searchable institutional knowledge that persists across projects and time.**

## CRITICAL: MEMORY DETAIL REQUIREMENTS

### Comprehensive Detail Mandate

**ALL MEMORIES MUST BE COMPREHENSIVE AND DETAILED**. Short, superficial memories are not acceptable. Each memory should be a complete reference document that future developers can use to understand and implement similar solutions.

### Required Detail Level

Every memory MUST include:

1. **Extensive Context** (3-5 sentences minimum)
   - What specific problem or situation led to this discovery
   - Why existing approaches weren't sufficient
   - What constraints or requirements influenced the solution

2. **Complete Implementation Details** (Code examples required)
   - Full code snippets showing the pattern or solution
   - Multiple examples when applicable (before/after, different scenarios)
   - File paths, function signatures, and configuration details
   - Migration scripts, SQL queries, or setup commands

3. **Comprehensive Technical Explanation**
   - How the solution works internally
   - Why this approach was chosen over alternatives
   - What makes this pattern effective or efficient
   - Technical trade-offs and considerations

4. **Measurable Impact** (Quantified when possible)
   - Performance improvements with specific metrics
   - User experience enhancements
   - Development efficiency gains
   - Error reduction or system stability improvements

5. **Future Application Guidance**
   - When to use this pattern vs alternatives
   - Warning signs or edge cases to watch for
   - How to adapt the pattern for different scenarios
   - Related patterns or concepts to consider

### Memory Length Expectation

- **Minimum**: 200-300 words per memory
- **Target**: 400-600 words with multiple code examples
- **Complex topics**: 800+ words with comprehensive coverage

## How to Store Memories

### Use the Memory CLI System

```bash
memories add --repo="repo name" --category="category" --tech_stack="{comma separated tech stacks}" --title="Your Title" --document="{inser full document here}"
```

### Handling Special Input

When using the command-line mode, especially for `document` field, you may need to include newlines or other special characters.

- **Newlines**: Use the special marker `__NEWLINE__` to represent a newline character. The script will automatically convert this to a proper newline (`\n`) when saving the memory.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [carlxrosales/cursor-memories](https://github.com/carlxrosales/cursor-memories) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
