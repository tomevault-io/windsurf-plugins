---
trigger: always_on
description: Handoff Message Templates, finish work proceduce
---

# Handoff Message Templates

This document contains standardized templates for all handoff messages in the SCRUM workflow. Use these templates for consistency and clarity when handing off work between roles.

## Feature Specification Complete
```
@SolutionArchitect

Task: [Task-ID]

Feature specification for [Feature Name] is complete and ready for architecture review.

Completed:
- Business requirements documented in [business.md](mdc:docs/business.md)
- User stories and acceptance criteria defined
- Integration points identified

Next steps:
- Architecture review needed
- Technical design to follow

Links:
- Feature documentation: docs/business.md#[feature-section]
- Related documents: docs/[related-doc].md
- Commit: [link-to-actual-commit]

Priority: [High/Medium/Low]
```

## Architecture Review Complete
```
@ScrumMaster

Task: [Task-ID]

Architecture review for [Feature Name] is complete and ready for sprint consideration.

Completed:
- Architecture decisions documented in [link-to-actual-docs]
- Component diagrams updated in [link-to-actual-docs]
- Technical constraints identified

Next steps:
- Sprint consideration needed
- Decision on current or future sprint inclusion required

Links:
- Architecture document: [link-to-actual-docs]
- Component diagrams: [link-to-actual-docs]
- Commit: [link-to-actual-commit]

Technical considerations:
- [Key technical considerations]
- Estimated technical complexity: [Low/Medium/High]
- Integration points with existing systems: [List key integration points]
```

## Architecture Review Decision
```
@ProductOwner @SolutionArchitect

Decision on architecture-reviewed task [Task-ID].

Decision:
- [ ] Included in current Sprint [X]
- [ ] Scheduled for next Sprint [X+1]
- [ ] Added to backlog for future consideration

Rationale:
- [Explain decision factors: team capacity, priority, dependencies, etc.]
- [Current sprint status if relevant]
- [Business or technical factors influencing decision]

Next steps:
- [If included] Technical design to begin immediately
- [If scheduled] Will be included in next sprint planning
- [If backlogged] Additional prioritization needed at next backlog refinement

Links:
- Updated sprint status: tasks/sprint-current-tasks.md
- Backlog with updated priority: tasks/backlog.md
- Related tasks: [links to dependent or related tasks]

Additional information:
- Estimated implementation effort: [Story points/days]
- Impact on current commitments: [Low/Medium/High]
- Resource requirements: [Any specific resource needs]
```

## Tasks Ready for Planning
```
@ScrumMaster

Task: [Task-ID]

Tasks for [Feature Name] are created and ready for sprint planning.

Completed:
- Task breakdown
- Acceptance criteria defined
- Dependencies identified
- Story points estimated

Next steps:
- Sprint planning consideration
- Priority assessment

Links:
- Task details: tasks/backlog.md#[task-id]
- Related tasks: [links to related tasks]
- Commit: [link-to-actual-commit]

Task Information:
- Estimated story points: [X]
- Dependencies: [List dependencies]
- Risk level: [Low/Medium/High]
```

## Task Selection for Sprint
```
@ProductOwner

Task selection for Sprint [X] is ready for review.

Completed:
- Task prioritization
- Capacity planning
- Dependency mapping
- Risk assessment

Proposed Tasks:
- [Task-ID-1]: [Brief description], [Story Points]
- [Task-ID-2]: [Brief description], [Story Points]
- [Task-ID-3]: [Brief description], [Story Points]
- [Optional Task-ID-4]: [Brief description], [Story Points]

Next steps:
- Your review and approval needed
- Sprint planning meeting scheduled for [date/time]
- Team capacity: [X] story points available

Links:
- Proposed sprint tasks: tasks/sprint-planning/sprint-[X]-proposal.md
- Task details: tasks/backlog.md
- Previous sprint metrics: tasks/sprint-metrics/sprint-[X-1].md

Selection Rationale:
- [Explanation of task selection criteria]
- [Notes on priorities and dependencies]
- [Capacity considerations]
```

## Sprint Planning Complete
```
@Team

Sprint planning for Sprint [X] is complete.

Completed:
- Sprint goals defined
- Tasks selected and prioritized
- Work capacity allocated
- Dependencies identified

Selected Tasks:
- [Task-ID-1]: [Brief description] - Assigned to @Developer1
- [Task-ID-2]: [Brief description] - Assigned to @Developer2
- [Task-ID-3]: [Brief description] - Assigned to @TestLeader

Next steps:
- Development work to begin
- Daily standups at [time]
- Sprint review scheduled for [date]

Links:
- Sprint planning document: tasks/sprint-planning/sprint-[X].md
- Sprint backlog: tasks/sprint-current-tasks.md
- Commit: [link-to-commit-with-sprint-planning-updates]

Sprint Goals:
- [Primary sprint objective]
- [Secondary sprint objective]
```

## Technical Design Request
```
@TechnicalLeader

Tasks for Sprint [X] are ready for technical design.

Tasks requiring design:
- [Task-ID-1]: [Brief description]
- [Task-ID-2]: [Brief description]
- [Task-ID-3]: [Brief description]

Sprint constraints:
- Design deadline: [Date - typically 2-3 days into sprint]
- Sprint end date: [Date]
- Implementation dependencies: [List any critical dependencies]

Architecture references:
- Feature architecture: [link-to-architecture-docs]
- System components: [link-to-component-docs]
- Design decisions: [link-to-design-decisions]


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reallongnguyen/nestjs-vibe-coding](https://github.com/reallongnguyen/nestjs-vibe-coding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
