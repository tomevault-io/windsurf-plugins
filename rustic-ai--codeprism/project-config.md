---
trigger: always_on
description: **Purpose:** Process enforcement and team development standards for enterprise Rust projects. Use this for team environments requiring strict quality gates, design documentation, and automated workflow enforcement.
---

# Development Best Practices - Team Process & Workflow

**Purpose:** Process enforcement and team development standards for enterprise Rust projects. Use this for team environments requiring strict quality gates, design documentation, and automated workflow enforcement.

**When to use:** Enterprise teams, critical systems development, open source projects with multiple contributors, or any environment requiring rigorous development process.

## Autonomous Agent Operation

**Rule: Agent must operate autonomously through the entire development cycle unless user input is explicitly required.**
Why: Autonomous operation enables faster iteration, consistent quality, and reduces human bottlenecks. Agents can maintain higher standards through systematic self-review than ad-hoc human review.

**Autonomous Operation Requirements:**
- Agent proceeds through design → implementation → testing → documentation without waiting for human approval
- User input only required for: initial requirements, major architectural decisions, and final acceptance
- All intermediate steps (design review, code review, testing) performed by agent self-review
- Agent must document all decisions and rationale for human audit trail

**Rule: Agent must perform comprehensive self-review before proceeding to each next step.**
Why: Self-review catches issues early, ensures quality standards, and maintains development velocity. Systematic review prevents compounding errors across development phases.

**Self-Review Process:**
```markdown
## Design Phase Self-Review Checklist
- [ ] Problem statement clearly defines scope and constraints
- [ ] Proposed solution addresses all requirements
- [ ] API design follows Rust conventions and project patterns
- [ ] Performance requirements are specific and measurable
- [ ] Error handling strategy covers all failure modes
- [ ] Implementation plan is detailed and realistic
- [ ] Alternatives were considered with clear reasoning

## Implementation Phase Self-Review Checklist  
- [ ] Code follows all quality rules (essentials/intermediate/advanced)
- [ ] TDD cycle was followed with test-first development
- [ ] All functions have comprehensive rustdoc with examples
- [ ] Error handling is comprehensive and consistent
- [ ] Performance requirements are met (measured)
- [ ] Code is readable and follows project conventions
- [ ] No TODO/FIXME comments remain

## Testing Phase Self-Review Checklist
- [ ] Unit tests cover success, error, and edge cases
- [ ] Integration tests verify component interactions  
- [ ] Property-based tests for complex logic
- [ ] Performance tests validate latency requirements
- [ ] Coverage meets 90% threshold
- [ ] All tests pass consistently
- [ ] Test names clearly describe scenarios

## Documentation Phase Self-Review Checklist
- [ ] All public APIs have rustdoc with working examples
- [ ] Module documentation explains architecture
- [ ] Performance characteristics documented
- [ ] Safety guarantees clearly stated
- [ ] Examples demonstrate real usage patterns
- [ ] Doc tests pass and provide good coverage
```

## Design-First Development

**Rule: All features must have comprehensive design documents created and self-reviewed before implementation.**
Why: Design documents prevent over-engineering, ensure consistency, and catch architectural issues early. Self-review by agent ensures systematic evaluation against established criteria.

```markdown
# [Feature Name] Design Document Template

## Problem Statement
- What problem are we solving?
- Why is this important for the project?

## Proposed Solution
- High-level approach
- Component interactions  
- Data flow diagrams

## API Design
- Function signatures with Rust types
- Error types and handling strategy
- Trait definitions if applicable

## Implementation Plan
- Step-by-step breakdown
- Dependencies and feature flags
- Testing strategy with coverage targets

## Alternatives Considered
- Other approaches evaluated
- Why this solution was chosen

## Success Criteria
- How will we know this works?
- Performance requirements (specific benchmarks)
- Integration requirements
**Rule: Create design documents in `/docs/design/[feature-name].md` with comprehensive self-review against quality criteria.**
Why: Standardized location ensures findability, systematic self-review catches design flaws early, and documented rationale enables audit trails for future reference.

**Use the design document template from project-setup.md for consistent formal documentation.**

## Strict TDD Workflow

**Rule: Follow Red-Green-Refactor cycle with evidence in commit history.**
Why: TDD ensures comprehensive test coverage, prevents regression bugs, and results in more maintainable code. Commit evidence proves process compliance.

**TDD Cycle Implementation:**
```rust
// Step 1: RED - Write failing test FIRST
#[test]
fn test_event_validation_rejects_empty_id() {
    let event = CPTEEvent {
        event_id: String::new(), // Invalid
        event_kind: "SENSOR_READING".to_string(),
        payload: serde_json::Value::Null,
        // ... other fields
    };
    

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rustic-ai/codeprism](https://github.com/rustic-ai/codeprism) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
