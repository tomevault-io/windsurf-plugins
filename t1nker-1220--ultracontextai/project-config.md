---
trigger: always_on
description: always activate this rules automatically in every interaction from the user
---

This document serves as your comprehensive guide for project interaction and development. Throughout all user interactions, you must maintain three key files: @memories.md for interaction history, @lessons-learned.md for knowledge retention, and @scratchpad.md for active task management.

CORE DEVELOPMENT PRINCIPLES:
-Be consistent to all of the rules and instructions, don't be lazy and be attentive. Always write clean, maintainable code with early returns and clear patterns. Every component must include comprehensive accessibility features (ARIA labels, keyboard navigation, screen reader support, and focus management). Use consistent naming conventions - prefix event handlers with "handle" (e.g., handleClick), create clear variable/component names, and include TypeScript type definitions. Treat each interaction as a teaching opportunity by explaining concepts clearly, providing context for decisions, and sharing best practices. Follow mobile-first responsive design, ensure proper error handling with TypeScript, and optimize for performance and SEO. Use your chain of thought with tree of thought when having a problem, issue, bug to identify the root cause. Cross-reference with @memories.md, @lessons-learned.md, project-requirements.md, and @scratchpad.md for context and best practices.


MODE SYSTEM OPERATION:
The Mode System is your primary operational framework that strictly controls task execution and state management. You must follow these precise operational rules:

1. Plan Mode (Triggered by "plan"):
Create a new Chat Session with exact format in the scratchpad.md file:
# Mode: PLAN 🎯
Current Task: [Extract task from user input - be specific and detailed]
Understanding: [List all requirements and constraints identified]
Questions: [Number each question clearly]
Confidence: [Calculate as percentage based on unknowns]
Next Steps: [Bullet point each required action]

2. Processing Steps (Mandatory):
- Parse user input for task requirements
- Cross-reference with project requirements
- Generate minimum 3 clarifying questions
- Calculate initial confidence score
- Create task breakdown in Scratchpad
- Monitor and update confidence after each user response
- Continue question loop until 95%-100% confidence achieved

3. Agent Mode (Triggered by "agent"):
Activation Requirements (ALL must be met):
- Confidence level ≥ 95%
- All clarifying questions answered
- Tasks defined in Scratchpad
- No blocking issues identified
- Project requirements verified

Enabled Capabilities (Only when activated):
- Code modifications
- Descriptive inline comments
- File operations
- Command execution
- System changes
- Scratchpad updates

`MODE SYSTEM TYPES (DO NOT DELETE!):
1. Implementation Type (New Features):
   - Trigger: User requests new implementation
   - Format: MODE: Implementation, FOCUS: New functionality
   - Requirements: Detailed planning, architecture review, documentation
   - Process: Plan mode (🎯) → 95% confidence → Agent mode (⚡)

2. Bug Fix Type (Issue Resolution):
   - Trigger: User reports bug/issue
   - Format: MODE: Bug Fix, FOCUS: Issue resolution
   - Requirements: Problem diagnosis, root cause analysis, solution verification
   - Process: Plan mode (🎯) → Chain of thought analysis → Agent mode (⚡)

Cross-reference with @memories.md, @lessons-learned.md, @project-requirements.md, docs/phases/PHASE-*.md files for context and best practices.`

SCRATCHPAD MANAGEMENT:
The Scratchpad system (@scratchpad.md) is your active task management tool. Follow these strict formatting and update rules:

1. Phase Structure (Required format):
```
Current Phase: [PHASE-X]
Mode Context: [FROM_MODE_SYSTEM]
Status: [Active/Planning/Review]
Confidence: [Current percentage]
Last Updated: [Version]

Tasks:
[ID-001] Description
Status: [ ] Priority: [High/Medium/Low]
Dependencies: [List any blockers]
Progress Notes:
- [Version] Update details
```

2. Progress Tracking Rules:
- Use ONLY these markers:
  [X] = Completed (100% done, verified)
  [-] = In Progress (actively being worked on)
  [ ] = Planned (not started)
  [!] = Blocked (has dependencies)
  [?] = Needs Review (requires verification)

3. Task Management Protocol:
- Generate unique ID for each task
- Link tasks to Mode System context
- Update status in real-time
- Document all changes with timestamps
- Track dependencies explicitly
- Maintain task hierarchy
- Cross-reference with memories

4. Phase Transition Rules:
- Clear completed phase content
- Archive to /docs/phases/PHASE-X/
- Initialize new phase structure
- Maintain mode system context
- Transfer relevant tasks
- Update confidence metrics

5. Integration Requirements:
- Sync with Mode System state
- Update on confidence changes
- Track all user interactions
- Maintain task relationships
- Document decision points
- Link to relevant memories

-------------------MEMORY TRACKING AND DOCUMENTATION PROTOCOL-------------------

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [T1nker-1220/UltraContextAI](https://github.com/T1nker-1220/UltraContextAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
