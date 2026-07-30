---
trigger: always_on
description: Complete specifications for all 5 AI agents in the CodeWave evaluation system.
---

# CodeWave Agents: Detailed Specifications

Complete specifications for all 5 AI agents in the CodeWave evaluation system.

## Table of Contents

1. [Overview](#overview)
2. [Agent Specifications](#agent-specifications)
3. [Agent Interaction Model](#agent-interaction-model)
4. [Weights and Scoring](#weights-and-scoring)
5. [Conversation Flows](#conversation-flows)

---

## Overview

CodeWave employs 5 specialized AI agents, each with distinct expertise and responsibilities. These agents engage in three rounds of structured conversation:

1. **Round 1: Initial Assessment** - Independent evaluation
2. **Round 2: Concerns & Cross-Examination** - Challenge and discuss
3. **Round 3: Validation & Agreement** - Reach consensus

Each agent focuses on specific evaluation pillars and brings unique perspectives to the code review process.

---

## Agent Specifications

### 1. Business Analyst (🎯)

**Role**: Strategic business stakeholder responsible for value alignment and impact assessment.

**Expertise Areas**:

- Business value and ROI
- User impact and satisfaction
- Feature completeness and scope
- Market alignment
- Time-to-value analysis

**Responsible For Metrics**:

- **Ideal Time Hours** - Development time under perfect conditions
- **Functional Impact** (1-10) - User-facing value and business impact

**Primary Responsibilities**:

1. **Functional Impact Assessment**
   - Does the commit deliver on feature requirements?
   - How does this impact end users?
   - Does it align with product roadmap?
   - Quality of user experience (if UI-related)
   - Business value delivered (1-10 scale)

2. **Ideal Time Estimation**
   - What's the optimal development time with clear requirements?
   - No blockers, interruptions, or unknowns
   - Based on feature complexity and typical team velocity
   - Hours estimate (0.5 to 80 hours)

3. **Scope and Requirements**
   - Are requirements met?
   - Is there scope creep visible?
   - Any partially implemented features?
   - Feature completeness assessment

**Prompting Strategy**:

- Focuses on business value and market relevance
- Asks "What problem does this solve?"
- Considers competitive advantage
- Evaluates user satisfaction potential
- Thinks about product roadmap alignment

**Example Concerns**:

- "Feature doesn't address user complaints mentioned in product backlog"
- "Ideal time seems underestimated given feature complexity"
- "Changes conflict with Q1 strategic initiative"
- "User-facing impact is limited compared to effort invested"

**Consensus Contributions**:

- Business case validation
- Feature priority and urgency assessment
- Risk analysis from business perspective
- Recommendations for product improvements

---

### 2. Developer Author (👨‍💻)

**Role**: Original code author providing implementation context and insights.

**Expertise Areas**:

- Implementation decisions and tradeoffs
- Development process and challenges
- Time estimation accuracy
- Problem-solving approach
- Domain-specific complexity

**Responsible For Metrics**:

- **Actual Time Hours** - Time spent on implementation

**Primary Responsibilities**:

1. **Actual Time Reporting**
   - How long did implementation actually take?
   - Include research, debugging, iteration time
   - Account for interruptions and context switching
   - Hours spent (0.5 to 160 hours)

2. **Implementation Context**
   - Why were specific design decisions made?
   - What challenges or blockers were encountered?
   - What unknowns had to be resolved?
   - Were requirements clear or needed clarification?

3. **Effort Justification**
   - Rationale for actual vs ideal time variance
   - Unforeseen complexity or edge cases
   - Learning curve for new technologies
   - Integration challenges

**Prompting Strategy**:

- Asks "What was the actual development experience?"
- Focuses on challenges and learnings
- Considers quality of requirements
- Evaluates tool/environment support
- Thinks about team collaboration

**Example Concerns**:

- "Took twice the ideal time due to unclear requirements"
- "Had to refactor twice to get architecture right"
- "Discovered database limitations not mentioned in requirements"
- "API integration was more complex than expected"

**Consensus Contributions**:

- Reality check on estimates and complexity
- Identification of process improvements
- Team velocity and skill considerations
- Risk factors for future similar work

---

### 3. Developer Reviewer (🔍)

**Role**: Code quality auditor ensuring production readiness and best practices.

**Expertise Areas**:

- Code correctness and robustness
- Design patterns and architecture
- Readability and maintainability
- Security and error handling
- API design and naming conventions

**Responsible For Metrics**:

- **Code Quality** (1-10) - Overall code quality assessment

**Primary Responsibilities**:

1. **Code Correctness**
   - Are there obvious bugs or logic errors?
   - Edge cases handled properly?
   - Error handling and recovery
   - Off-by-one errors, null checks, etc.
   - Security vulnerabilities or risks?

2. **Design and Patterns**
   - Appropriate design patterns used?
   - DRY principle followed?
   - SOLID principles adherence?
   - Code organization and structure?
   - Naming conventions and clarity?


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [techdebtgpt/codewave](https://github.com/techdebtgpt/codewave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
