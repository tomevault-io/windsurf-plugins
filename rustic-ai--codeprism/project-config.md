---
trigger: always_on
description: **Purpose:** Specific workflow rules for AI/SWE agents to ensure proper integration with development tools, processes, and quality gates. These rules govern agent behavior when working autonomously on software development tasks.
---

# Agent Workflow Rules - SWE Agent Compliance

**Purpose:** Specific workflow rules for AI/SWE agents to ensure proper integration with development tools, processes, and quality gates. These rules govern agent behavior when working autonomously on software development tasks.

**When to use:** AI agents, automated development workflows, SWE bots, or any autonomous system performing development tasks.

## Work Organization and Planning

**Rule: Create temporary files in the cursor_workspace folder**
Why: Creaitng files under /tmp requires user aproaval and breaks the flow, so create them in a folder in workspace that is not commited to git.

**Rule: Organize work as milestones, phases, and priorities - never add timelines, schedules, or estimates.**
Why: Agile development focuses on value delivery over time predictions. Estimates are often inaccurate and create false expectations. Priorities and milestones provide clear direction without artificial time constraints.

**Implementation:**
```markdown
# ✅ GOOD - Use milestones and priorities
## Milestone 1: Core Authentication
- High Priority: JWT token validation
- Medium Priority: Session management  
- Low Priority: Multi-factor authentication

## Phase 2: User Management
- User creation and validation
- Role-based access control
- Account management features

# ❌ BAD - Avoid timelines and estimates  
## Week 1 (Est. 15 hours)
- [ ] JWT implementation (5 hours)
- [ ] Session management (8 hours)
- [ ] Testing (2 hours)
```

## Git Workflow Compliance

**Rule: After completing work on an issue make sure all the changes are committed and there are no staged or untracked changes**
Why: The work done is in the files and it is essential to commit the files to git. Untracked, staged artifacts left behind will mean that our project will not be functional. 

**Rule: Never bypass git pre-commit hooks - always run validation before committing.**
Why: Pre-commit hooks maintain code quality and catch issues early. Bypassing them, even for documentation, can introduce inconsistencies and break CI/CD pipelines.

**Enforcement:**
- **NEVER use:** `git commit --no-verify`
- **NEVER use:** `git commit -n` 
- **ALWAYS run:** Standard `git commit` to trigger pre-commit validation
- **Applies to:** All commits including docs-only, formatting fixes, or minor changes

```bash
# ✅ ALWAYS do this
git add .
git commit -m "docs: update API documentation"

# ❌ NEVER do this - even for docs
git commit --no-verify -m "docs: quick fix"
git commit -n -m "format: minor spacing"
```

**Rule: Always push your changes upstream, once you have a successful commit.**
Why: Pushing changes upstream makes sure that the changes are available for others to work with, safe and protected from system failures and also closes the relevant issues on GitHub automatically.

## Issue Resolution Verification

**Rule: Before committing, verify the change completely fixes the issue and meets all requirements.**
Why: Incomplete solutions create technical debt, confuse stakeholders, and require additional work cycles. Thorough verification prevents rework and ensures deliverable quality.

**Pre-commit Verification Checklist:**
```markdown
Before committing any issue fix:
- [ ] Read the original issue requirements completely
- [ ] Verify each requirement is addressed in the code
- [ ] Test the specific scenarios mentioned in the issue
- [ ] Ensure no placeholder code or TODOs remain
- [ ] Verify error cases and edge conditions are handled
- [ ] Check that related functionality still works
- [ ] Confirm the fix doesn't introduce new issues
```

**Code Review Questions:**
- Does this change solve the exact problem described?
- Are all acceptance criteria met?
- Would someone reviewing this code understand the complete solution?
- Are there any stubs, TODOs, or incomplete implementations?

## Design-First Development

**Rule: Write detailed solution/design in issue comment before starting work - stick to it or update it.**
Why: Solution comments serve as implementation contracts, prevent scope creep, and provide audit trails. They complement the formal design documents from dev-best-practices.md with issue-specific implementation plans.

**Solution Comment Template (for GitHub issue comments):**
```markdown
## Solution Design

### Problem Analysis
- Root cause of the issue
- Impact on existing functionality
- Requirements and constraints

### Proposed Approach
```rust
// High-level API design
pub struct UserManager {
    repository: Arc<dyn UserRepository>,
    validator: UserValidator,
}

impl UserManager {
    pub async fn create_user(&self, request: CreateUserRequest) -> Result<User, UserError> {
        // Implementation approach
    }
}
```

### Implementation Steps
1. **Phase 1**: Core data structures and validation
2. **Phase 2**: Database integration and persistence
3. **Phase 3**: API endpoints and error handling
4. **Phase 4**: Testing and documentation

### Testing Strategy
- Unit tests for validation logic
- Integration tests for database operations
- End-to-end tests for API workflows

### Breaking Changes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rustic-ai/codeprism](https://github.com/rustic-ai/codeprism) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
