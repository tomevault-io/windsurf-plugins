---
trigger: always_on
description: Guidelines for identifying when a user story should be split into smaller, more manageable stories
---


# Story Splitting Rule

## Purpose & Scope

This rule defines criteria and patterns for recognizing when a user story is too large and should be split into multiple smaller stories. Proper story splitting enables better manageability, risk reduction, delivery predictability, and team velocity. It helps teams deliver value incrementally and maintain sustainable sprint commitments.

**Applies to**: All User Stories during backlog grooming, sprint planning, or mid-sprint refinement where story size, complexity, or risk is assessed.

**Does not apply to**: Epics (already designed to be split into Features/Stories), appropriately-sized stories (≤8 points), trivial tasks (<1 point), or stories that would lose cohesion if split further.

## When to Consider Story Splitting

### Size Indicators (Any of These)
- **Story Points > 8**: Stories larger than 8 points have high uncertainty and risk
- **Multiple Epics Worth**: Story spans scope that could justify multiple user stories
- **Cannot Complete in One Sprint**: Story cannot be fully implemented and tested within iteration timeframe
- **Multiple Major Components**: Story touches 3+ major components/systems
- **Conversation Explosion**: During planning, conversations balloon with "and also we need..." repeatedly

### Complexity Indicators (Any of These)
- **Multiple Critical Paths**: Story has several independent critical paths that could be delivered separately
- **Unclear Requirements**: After initial analysis, requirements remain vague or require extensive discovery
- **High Risk Areas**: Story contains multiple high-risk areas that should be tackled incrementally
- **Multiple Acceptance Criteria Sets**: Acceptance criteria naturally group into distinct functional areas
- **Implementation Spans Multiple Layers**: Frontend + Backend + Database + Integration all in one story

### Dependency Indicators (Any of These)
- **Sequential Dependencies**: Story has clear sequential phases that could be independent stories
- **Blocking Multiple Teams**: Story blocks work for multiple teams/developers who could work in parallel
- **External Dependencies**: Story depends on multiple external systems/teams with different timelines
- **Infrastructure + Feature**: Story mixes infrastructure setup with feature delivery

### Risk Indicators (Any of These)
- **Unknown Unknowns**: Significant technical uncertainty that needs investigation before commitment
- **Multiple Failure Modes**: Too many ways the story could fail or need rework
- **All-or-Nothing Delivery**: No way to deliver incremental value if story fails mid-implementation
- **Testing Complexity**: Testing strategy is unclear or requires extensive test scenario development

## Story Splitting Patterns

### Pattern 1: By User Journey Steps
Split based on user workflow stages:
- **Before**: "User can complete entire purchase flow"
- **After Split**:
  - Story 1: User can add items to cart
  - Story 2: User can proceed to checkout
  - Story 3: User can complete payment
  - Story 4: User receives order confirmation

### Pattern 2: By Technical Layer
Split based on system layers:
- **Before**: "Implement user authentication"
- **After Split**:
  - Story 1: Implement authentication data model and database
  - Story 2: Implement authentication API endpoints
  - Story 3: Implement authentication UI components
  - Story 4: Integrate authentication with existing features

### Pattern 3: By Business Rule Complexity
Split based on business rule complexity:
- **Before**: "Implement pricing calculation with all rules"
- **After Split**:
  - Story 1: Implement base pricing calculation
  - Story 2: Add volume discount rules
  - Story 3: Add promotional pricing rules
  - Story 4: Add customer-specific pricing rules

### Pattern 4: By Happy Path vs Edge Cases
Split based on scenario complexity:
- **Before**: "Implement file upload with all scenarios"
- **After Split**:
  - Story 1: Implement basic file upload (happy path)
  - Story 2: Handle file validation errors
  - Story 3: Handle large file uploads
  - Story 4: Handle network interruption recovery

### Pattern 5: By Infrastructure vs Feature
Split setup work from feature delivery:
- **Before**: "Implement notification system"
- **After Split**:
  - Story 1: Setup notification infrastructure and templates
  - Story 2: Implement email notifications
  - Story 3: Implement SMS notifications
  - Story 4: Implement in-app notifications

### Pattern 6: By Integration Points
Split based on external dependencies:
- **Before**: "Integrate with payment providers"
- **After Split**:
  - Story 1: Integrate with primary payment provider
  - Story 2: Integrate with backup payment provider
  - Story 3: Implement payment provider fallback logic
  - Story 4: Add payment provider monitoring

### Pattern 7: By Validation Phases
Split based on validation complexity:
- **Before**: "Implement temporary structure builder with validation"
- **After Split**:
  - Story 1: Design and implement bottom-up structure builder
  - Story 2: Implement LAYER field marker logic
  - Story 3: Implement atomic topnode swap mechanism
  - Story 4: Implement structure validation and cleanup

## Splitting Decision Framework

### Step 1: Initial Assessment
Ask these questions:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ai-Whisperers) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
