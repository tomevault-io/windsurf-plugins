---
trigger: always_on
description: Enforces structured development workflow using GitHub issues, milestones, and projects. Ensures systematic progression through milestone-focused development with comprehensive tracking and verification.
---

# GitHub Development Workflow Rules

**Purpose:** Enforces structured development workflow using GitHub issues, milestones, and projects. Ensures systematic progression through milestone-focused development with comprehensive tracking and verification.

**When to use:** All development work in GitHub repositories, milestone-based projects, systematic feature development, and any work requiring complete traceability.

## Milestone-Focused Development

**Rule: Work on ONE milestone at a time from start to complete finish.**
Why: Focused development ensures functional releases, prevents context switching, and maintains clear progress tracking. Milestones represent complete functional value delivery.

**Milestone Requirements:**
- **Start Only One**: Never work on multiple milestones simultaneously
- **Complete ALL Issues**: Milestone is done only when 100% of issues are closed
- **No Partial Releases**: Don't close milestone until every issue is resolved
- **Emergency Exception**: Only P0 critical bugs can interrupt current milestone

**Milestone Selection Process:**
```bash
# Check current milestone status
gh issue list --milestone "current-milestone" --state open

# If no open issues in current milestone, select next milestone
# If issues remain, continue working on current milestone
```

## Issue Discovery and Creation

**Rule: Immediately create new issues when discovering bugs, limitations, or missing work during development.**
Why: Comprehensive tracking prevents lost work, maintains milestone completeness, and provides audit trail for all discovered requirements.

**Discovery Triggers - Create New Issue When:**
- **Bug Found**: Code doesn't work as expected during testing
- **Design Limitation**: Current approach won't achieve requirements  
- **Missing Functionality**: Additional work needed for completion
- **Performance Issue**: Implementation doesn't meet requirements
- **Integration Problem**: Feature doesn't work with existing code
- **Documentation Gap**: Missing or incorrect documentation

**New Issue Creation Process:**
```bash
# Stop current work, create issue immediately
gh issue create --title "Bug: [description]" \
  --body "Discovered while working on #123
  
## Problem
[Description of discovered issue]

## Impact  
[How this affects original work]

## Required Action
[What needs to be done]" \
  --label "found-during-dev,bug,P1" \
  --milestone "current-milestone"

# Link to original issue in description
# Assign appropriate priority based on impact
# Always assign to current milestone
```

**Issue Linking Requirements:**
- **Reference Original**: "Discovered while working on #123"
- **Explain Impact**: How discovery affects original work
- **Set Priority**: Based on blocking impact on milestone
- **Assign to Current Milestone**: Keep all related work together

## Documentation-First + TDD Workflow

**Rule: Every issue must follow Documentation-First + TDD workflow before any implementation.**
Why: Documentation-first ensures clear requirements understanding, TDD ensures robust implementation, and this combination prevents rework and missing requirements.

**Mandatory Workflow Sequence:**
```markdown
1. **Documentation Phase** (before any code)
   - Write/update design documentation for the issue
   - Document expected API and behavior
   - Define acceptance criteria clearly
   - Update user documentation if needed

2. **TDD Phase** (before implementation)  
   - Write failing tests first (red phase)
   - Write minimal code to pass tests (green phase)
   - Refactor code while keeping tests green
   - Repeat until feature complete

3. **Implementation Phase**
   - Complete implementation following TDD cycle
   - Maintain test coverage ≥90%
   - Follow code quality rules from rust-essentials.md

4. **Verification Phase** (mandatory before completion)
   - Run complete verification checklist
   - Create additional issues if problems discovered
   - Only proceed to commit after verification passes

5. **Commit & Push Phase** (mandatory after verification)
   - Commit all changes with proper issue linking
   - Push changes to upstream repository
   - Ensure all work is saved and available

6. **Issue Completion**
   - Mark issue as done only after commit/push complete
   - Update issue status to reflect completion
```

**Documentation Requirements:**
- **API Documentation**: Every public function has rustdoc with examples
- **User Documentation**: Feature usage and integration guides
- **Design Documentation**: Technical approach and architecture decisions
- **Test Documentation**: Test strategy and coverage rationale

## Task Completion Verification

**Rule: No issue can be marked as 'done' without completing the full verification checklist.**
Why: Systematic verification prevents incomplete work, ensures quality standards, and catches integration issues before they compound.

**Mandatory Verification Checklist:**
```markdown
## Task Completion Verification (Required Before Done)

### Requirements Verification
- [ ] Re-read original issue requirements completely
- [ ] Verify each requirement is implemented and working

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rustic-ai/codeprism](https://github.com/rustic-ai/codeprism) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
