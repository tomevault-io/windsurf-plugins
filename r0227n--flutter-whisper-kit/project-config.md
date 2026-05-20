---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Documentation Structure and Relationship

### Language Configuration

- **Default Language**: Japanese
- **README.md Language**: The README.md file uses the default language setting for all documentation
- **Translation Support**: Multi-language documentation can be provided through the i18n system

### Mermaid Visualization Workflow

The README.md file includes Mermaid diagrams to visualize:

- Project architecture overview
- Development workflow processes
- Component relationships
- System integration patterns

## Claude 4 Best Practices Implementation

This project follows the Claude 4 prompt engineering best practices defined in `docs/CLAUDE_4_BEST_PRACTICES.md`. The implementation focuses on the core principle:

**「小さなドラフト → 厳しい批評 → 再生成 → リリース」(Small draft → Critical review → Regenerate → Release)**

### Core Implementation Principles

1. **AI Review-First Design**: Use Claude as a "Senior Reviewer" rather than "Junior Designer"
2. **Structured Quality Assessment**: Security (HIGH) → SOLID Principles (MEDIUM) → Performance (LOW)
3. **Iterative Review Cycles**: 3-4 review iterations maximum with 400-character summaries
4. **Human Final Validation**: Always include human verification as the final step

For comprehensive details, refer to [Claude 4 Best Practices](docs/CLAUDE_4_BEST_PRACTICES.md).

### AI Review-First Implementation Workflow

Implement the core Claude 4 Best Practices workflow for all development tasks:

#### Step 1: Minimal Implementation Draft

Create basic functionality with essential features:

```text
Context: Flutter monorepo with Riverpod + go_router + slang
Task: [Specific feature name]
Requirements:
- Essential functionality only
- Follow existing project patterns
- Use established dependencies and conventions
Output: Minimal working implementation
```

#### Step 2: AI Review Cycles (3-4 iterations)

**Review Template for All Code**:

```text
Review Request: [Brief description]

Apply Claude 4 AI Review-First methodology:

1. Security Analysis (HIGH Priority):
   - Hardcoded secrets/credentials scan
   - Input validation and sanitization
   - Secure data storage practices
   - Network communication security

2. Architecture Review (MEDIUM Priority):
   - SOLID principles compliance
   - Design pattern consistency
   - Separation of concerns
   - Code organization and structure

3. Performance Review (LOW Priority):
   - Bottleneck identification
   - Algorithm efficiency
   - Resource utilization
   - Build/runtime optimization

Format: Max 400 characters per category
Output: Specific actionable recommendations
Priority: Mark each issue as HIGH/MEDIUM/LOW
```

#### Step 3: Quality Gate Validation

All implementations must pass these gates:

```bash
# Security Gates (HIGH Priority)
- No hardcoded secrets: grep -r --exclude-dir=.git -E "(API_KEY|SECRET|PASSWORD|TOKEN|PRIVATE_KEY)" .
- Input validation implemented
- Secure storage patterns used

# Architecture Gates (MEDIUM Priority)
mise run analyze          # Static analysis
mise run test            # All tests pass

# Performance Gates (LOW Priority)
- No obvious bottlenecks identified
- Efficient patterns used
```

#### Step 4: Final Validation Checklist

- [ ] HIGH priority security issues: ✅ Resolved
- [ ] MEDIUM priority architecture issues: ✅ Addressed
- [ ] LOW priority performance issues: ✅ Optimized
- [ ] Static analysis: ✅ `mise run analyze` passes
- [ ] All tests: ✅ `mise run test` passes
- [ ] Translation validation: ✅ `mise run analyze-slang` passes
- [ ] Code formatting: ✅ `mise run format` applied
- [ ] Human validation: ✅ Final review completed

### Context Requirements for AI Interactions

When working with this codebase, always provide:

1. **Project Context**: This is a Flutter monorepo using Melos for package management
2. **Architecture Context**: Riverpod for state management, go_router for navigation, slang for i18n
3. **Code Generation**: Uses build_runner, requires running `melos run gen` after model changes
4. **Testing Strategy**: Widget tests in `app/test/`, run via `melos run test`
5. **Quality Assurance**: Multi-stage pipeline with static analysis, formatting, and translation validation

### Effective Prompting Guidelines

#### For Feature Implementation (AI Review-First)

Use this Claude 4 optimized prompt structure:

```text
Phase 1 - Minimal Implementation:
Context: Flutter monorepo with Riverpod + go_router + slang architecture
Task: Implement [specific feature name]
Constraints:
- Essential functionality only (no extra features)
- Reference existing patterns: [provide specific file examples]
- Follow project conventions: [naming, structure, dependencies]
- Technology stack: Riverpod (@riverpod), go_router (type-safe), slang (ja/en)

Phase 2 - AI Review Request:
Apply Claude 4 Review-First methodology:
1. Security scan (HIGH): secrets, validation, storage
2. Architecture review (MEDIUM): SOLID principles, patterns
3. Performance check (LOW): bottlenecks, efficiency
Format: 400 chars max per category, actionable recommendations

Phase 3 - Quality Validation:
- Pass: mise run analyze (static analysis)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [r0227n/flutter_whisper_kit](https://github.com/r0227n/flutter_whisper_kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
