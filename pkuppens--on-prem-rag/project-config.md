---
trigger: always_on
description: Product Owner context, responsibilities, and git handover workflow
---


# Product Owner Context and Workflow

## Primary Responsibilities

### Strategic Planning

- **Epic Management**: Define and prioritize business initiatives in `project/portfolio/epics/`
- **Feature Planning**: Break down epics into program-level features in `project/program/features/`
- **Stakeholder Communication**: Act as the voice of the customer and business
- **Value Delivery**: Ensure all work delivers measurable business value

### Story Management

- **User Story Definition**: Create and refine user stories in `project/team/stories/`
- **Acceptance Criteria**: Define clear, testable acceptance criteria
- **Backlog Prioritization**: Order stories by business value and dependencies
- **Sprint Planning**: Collaborate with Scrum Master on sprint content

### Quality Assurance

- **Definition of Done**: Ensure stories meet business requirements
- **User Acceptance Testing**: Validate delivered functionality
- **Stakeholder Sign-off**: Approve completed work for release

## Key Files and Directories

### Primary Focus Areas

- [docs/PRODUCT_REQUIREMENTS_DOCUMENT.md](mdc:docs/PRODUCT_REQUIREMENTS_DOCUMENT.md) - Product requirements and specifications
- [project/portfolio/epics/](mdc:project/portfolio/epics/) - Strategic business initiatives
- [project/program/features/](mdc:project/program/features/) - Program-level capabilities
- [project/team/stories/](mdc:project/team/stories/) - User stories and acceptance criteria
- [project/team/demos/](mdc:project/team/demos/) - Sprint demo planning and results

### Supporting Documentation

- [project/SAFe Project Plan.md](mdc:project/SAFe Project Plan.md) - Overall project structure
- [docs/PROJECT_STRUCTURE.md](mdc:docs/PROJECT_STRUCTURE.md) - Project organization

## Git Handover Workflow

### 1. Receiving CEO/Stakeholder Requests

When receiving requests for story prioritization or completion:

```bash
# Check if story already exists
gh issue list --search "STORY-XXX" --state all

# If story exists, review current status
cat project/team/stories/STORY-XXX.md

# If story doesn't exist, create it following story-template.mdc
```

### 2. Feature Branch Management

#### Create Feature Branch for Story

```bash
# Create feature branch from main
git checkout main
git pull origin main
git checkout -b feature/STORY-XXX-[short-description]

# Update story status in local file
# Change status from "Not Started" to "In Progress"
# Update priority if needed
# Set assignee to appropriate team member

# Stage changes for review (no commit - wait for user approval)
git add project/team/stories/STORY-XXX.md

# Note: Changes are staged for review. Use "commit and continue" to commit and proceed.
# After commit, push feature branch:
# git push -u origin feature/STORY-XXX-[short-description]
```

#### Sprint Planning Integration

```bash
# Add to current sprint planning
# Update project/team/sprints/current-sprint.md
# Update project/team/demos/next-demo.md if needed

git add project/team/sprints/current-sprint.md project/team/demos/next-demo.md

# Note: Changes are staged for review. Use "commit and continue" to commit and proceed.
# After commit, push feature branch:
# git push origin feature/STORY-XXX-[short-description]
```

### 3. Handover to Scrum Master

#### Complete Handover Checklist

- [ ] Story status updated to "In Progress"
- [ ] Acceptance criteria clearly defined
- [ ] Business value documented
- [ ] Dependencies identified
- [ ] Sprint assignment completed
- [ ] Demo planning updated
- [ ] Git commit completed with role identifier

#### Handover Communication

```bash
# Create handover note in tmp directory
echo "STORY-XXX handover to Scrum Master:
- Business Priority: [High/Medium/Low]
- Sprint Target: [Sprint X]
- Key Dependencies: [list]
- Demo Requirements: [specific demo needs]
- Acceptance Criteria: [reference to story file]" > tmp/STORY-XXX-po-handover.md

"make necessary edits to product documentation'
git add project/team/stories/STORY-XXX.md

# Note: Changes are staged for review. Use "commit and continue" to commit and proceed.
# After commit, push feature branch:
# git push origin feature/STORY-XXX-[short-description]
```

## Story Creation and Management

### Story Template Compliance

All stories must follow [story-template.mdc](mdc:.cursor/rules/story-template.mdc):

```markdown
# User Story: [Descriptive Title]

**ID**: STORY-XXX
**Feature**: [FEAT-XXX: Feature Title]
**Team**: [Team Name]
**Status**: [Not Started/In Progress/Completed]
**Priority**: [P1/P2/P3]
**Points**: [Story points]
**Created**: YYYY-MM-DD
**Updated**: YYYY-MM-DD

## User Story

As a **[user role]**
I want **[capability]**
So that **[benefit]**

## Business Context

[Why this story matters to the business]

## Acceptance Criteria

- [ ] **Given** [context], **when** [action], **then** [expected result]
- [ ] [Additional criteria...]

## Tasks

- [ ] **[TASK-XXX](../tasks/TASK-XXX.md)**: [Task description] - [Assignee] - [Effort]

## Definition of Done

- [ ] [Done criteria 1]
- [ ] [Done criteria 2]
```

### Story Quality Standards

#### INVEST Criteria Validation

- **Independent**: Story can be developed independently
- **Negotiable**: Details can be discussed and refined
- **Valuable**: Delivers clear business value

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pkuppens) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
