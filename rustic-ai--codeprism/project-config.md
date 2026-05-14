---
trigger: always_on
description: **ALWAYS follow Design-First Development for every feature and bug fix.**
---

# CodePrism Development Workflow Rules

## Design-First Development

**ALWAYS follow Design-First Development for every feature and bug fix.**

### Process:
1. **Create design document first** in `design-docs/` folder before any implementation
2. **Post design as comment** on the corresponding GitHub issue
3. **Get design approval** before starting implementation
4. **Follow design exactly** or update design document if changes needed

### Design Document Structure:
```markdown
# [Feature/Issue Name] Design Document

## Problem Statement
- Clear description of what needs to be solved
- Why this is important for the project

## Proposed Solution
- High-level approach and architecture
- Component interactions and data flow
- API design with Rust types

## Implementation Plan
- Step-by-step breakdown
- Dependencies and integration points
- Testing strategy

## Alternatives Considered
- Other approaches evaluated
- Rationale for chosen solution

## Success Criteria
- How to verify the solution works
- Performance requirements
- Integration requirements
```

### File Naming Convention:
- `design-docs/[issue-number]-[descriptive-name].md`
- Example: `design-docs/042-event-validation-system.md`

## Test-Driven Development (TDD)

**ALWAYS follow TDD cycle: Red → Green → Refactor**

### TDD Process:
1. **RED**: Write failing test first
2. **GREEN**: Write minimal code to make test pass
3. **REFACTOR**: Improve code quality while keeping tests green
4. **Repeat**: Continue cycle for each new piece of functionality

### Test Requirements:
- Unit tests for all business logic
- Integration tests for component interactions
- Property-based tests for complex algorithms
- Performance tests for critical paths
- Minimum 90% code coverage

### Test Structure:
```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_feature_success_case() {
        // Arrange
        let input = create_valid_input();
        
        // Act
        let result = function_under_test(input);
        
        // Assert
        assert!(result.is_ok());
        assert_eq!(result.unwrap(), expected_output);
    }
    
    #[test]
    fn test_feature_error_case() {
        // Test error conditions
    }
    
    #[test]
    fn test_feature_edge_cases() {
        // Test boundary conditions
    }
}
```

## Pre-Commit Verification

**ALWAYS verify changes before committing using systematic review process.**

### Pre-Commit Checklist:
1. **Generate changelog** from staged changes
2. **Compare against issue requirements** - ensure all requirements addressed
3. **Compare against design document** - ensure implementation follows design
4. **Run complete test suite** - all tests must pass
5. **Verify no placeholder code** - no TODO/FIXME/unimplemented!() allowed
6. **Check code coverage** - must maintain 90% minimum
7. **Run formatting and linting** - cargo fmt && cargo clippy

### Change Verification Process:
```bash
# 1. Generate changelog from staged changes
git diff --cached --name-only > changed_files.txt
git diff --cached > changes.patch

# 2. Review changes against issue requirements
# - Read GitHub issue thoroughly
# - Verify each requirement is addressed
# - Check that no requirements are missed

# 3. Review changes against design document
# - Ensure implementation follows design
# - Verify API matches design specifications
# - Check that architecture is maintained

# 4. Run comprehensive verification
cargo test --all-features
cargo clippy --all-targets --all-features -- -D warnings
cargo fmt --all -- --check
```

### Commit Message Format:
```
type(scope): brief description

- Detailed description of changes
- How it addresses the issue requirements
- Design document reference: design-docs/[issue-number]-[name].md
- Tests: [number] tests added/updated, all passing
- Coverage: [percentage]% maintained

closes #[issue-number]
```

## GitHub Integration

### Common Commands:
```bash
# List issues
gh issue list | cat

# View specific issue
gh issue view [issue-number] | cat

# Create issue comment with design
gh issue comment [issue-number] --body-file design-docs/[issue-number]-[name].md

# View pull requests
gh pr list | cat

# View milestones
gh api repos/:owner/:repo/milestones | cat
```

### GitHub Workflow:
1. **Issue Assignment**: Use `gh issue view [number]` to understand requirements
2. **Design Posting**: Use `gh issue comment` to post design document
3. **Progress Updates**: Use `gh issue comment` for status updates
4. **Issue Closure**: Use "closes #[number]" in commit messages for auto-closure

## Workflow Integration

### Complete Development Cycle:
1. **Issue Analysis**: Read issue with `gh issue view [number] | cat`
2. **Design Creation**: Create design document in `design-docs/`
3. **Design Posting**: Post to issue with `gh issue comment`
4. **TDD Implementation**: Follow Red-Green-Refactor cycle
5. **Pre-Commit Verification**: Systematic review against requirements and design
6. **Commit with Closure**: Use proper commit message format with "closes #[number]"

### Key Project Files:
- Implementation completeness checker: [scripts/check-implementation-completeness.sh](mdc:scripts/check-implementation-completeness.sh)
- MCP Specification: [specification/](mdc:specifications/)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rustic-ai/codeprism](https://github.com/rustic-ai/codeprism) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
