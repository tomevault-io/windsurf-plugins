---
trigger: always_on
description: Dynamic phase-based instruction loading for token-efficient development.
---

# Archflow — Phase-Based Development Instructions
Dynamic phase-based instruction loading for token-efficient development.

## 🎯 Core Agents (Always Available)

**Phase 1: Strategy & Planning**
- `product-strategist` - Business strategy, personas, KPIs → .archflow/project-context.md
- `feature-planner` - Feature roadmaps, user stories, sprint planning → .archflow/roadmap.yaml

**Phase 2: Design**
- `ux-designer` - User flows, design systems, themes, wireframes → design-artifacts/
- `dsl-generator` - Component specifications with styling → design-artifacts/styled-dsl.yaml

**Phase 2.25: High-Fidelity Design**
- SuperDesign MCP - Hi-fi screen generation from styled-dsl.yaml → design-artifacts/hifi-screens/

**Phase 2.5: API Architecture**
- `api-contract-architect` - API contracts from wireframes → docs/api-contract.md (single source of truth)

**Phase 3: Implementation**
- `ui-engineer` - All frontend (React, React Native, SwiftUI, Jetpack Compose) + integration with backend APIs. Also updates screens after ux-designer changes `styled-dsl.yaml`
- `api-engineer` - NestJS/PostgreSQL backends, MUST follow docs/api-contract.md exactly (zero tolerance)
- `qa-engineer` - Comprehensive testing (unit, integration, e2e) across all platforms. Runs AFTER feature agents complete
- `pm-maestro-reviewer` - Acceptance testing via Maestro. Runs AFTER qa-engineer, validates acceptance criteria from .archflow/roadmap.yaml → docs/acceptance-reports/
- `ux-designer` - Design updates on specific screens. Updates `styled-dsl.yaml` file

**Phase 4: Quality & Optimization**
- `code-reviewer` - Code quality, security, best practices analysis + improvement reports
- `performance-optimizer` - Performance bottleneck identification and optimization
- `pm-maestro-reviewer` - Full acceptance regression suite across all implemented features

**Phase 5: Launch & Operations**
- `devops-engineer` - CI/CD pipelines, deployment infrastructure, app store preparation
- `post-launch-analyst` - Analytics implementation, user insights, performance monitoring

**Phase 6: Enhancement (On-Demand)**
- `i18n-engineer` - Internationalization for web/iOS/Android platforms

## 📌 Available Commands (Archflow)
- `/archflow` — Show available subcommands and current project status
- `/archflow init` — Initialize Archflow in a new project (creates `.archflow/` state files, sets Phase 1)
- `/archflow onboard` — Onboard existing codebase (interactive wizard: audit, import context, backfill artifacts, set phase)
- `/archflow setup-mcp` — Configure an MCP server for external tools (Jira, Notion, Linear, GitHub, SuperDesign, etc.)
- `/archflow feature` — Add a new feature to the roadmap and start the git development workflow

## 🌐 Project Types
The framework detects and adapts to project type: `fullstack`, `frontend_only`, `backend_only`, `mobile`.
- Stored in `.archflow/current-phase.yaml` as `project_type`
- Phases, agents, and audit checks are filtered by project type
- `.archflow/roadmap.yaml` structure is tailored to project type (backend = endpoints/services, frontend = pages/components)
- Set automatically by `/archflow onboard` or can be set manually in `.archflow/current-phase.yaml`

## 🔀 Git Workflow
All feature development follows `.archflow/workflow.md` branching strategy:
- Feature branches from main
- Task branches from feature
- Subtask branches from task
- Merge only after explicit user approval
- Use `/archflow feature` to create feature branches and track tasks in `.archflow/current-feature.yaml`

## 🔄 Dynamic Phase Loading

**Current Phase Detection:**
```yaml
# Read .archflow/current-phase.yaml to determine active phase
phase: 1  # Current phase number
phase_file: ".archflow/phases/phase-1-strategy.md"  # Load this file for detailed instructions
```

**Phase Instruction Loading:**
```bash
# Check for existing phase tracker
if [[ -f ".archflow/current-phase.yaml" ]]; then
  # Normal operation - use existing tracker
  Current Phase: .archflow/current-phase.yaml → phase_file
  Detailed Instructions: .archflow/phases/phase-{current}-{name}.md
else
  # Project setup needed - load setup system
  Setup Required: .archflow/phases/phase-setup.md → detect and initialize phase
fi
```

## 📋 Universal Context Files (Always Required)
- `.archflow/project-context.md` - Business goals, tech stack, architecture decisions
- `.archflow/roadmap.yaml` - Feature roadmap and sprint planning
- `.archflow/current-feature.yaml` - Active development scope and requirements
- `.archflow/current-phase.yaml` - Phase state tracker (PROJECT-SCOPED, auto-created from template)

## 💡 Universal Critical Rules (Apply to ALL Phases)

### 🚨 API Contract Compliance (Phases 2.5-4)
- **API CONTRACT IS SACRED**: api-engineer AND ui-engineer MUST follow docs/api-contract.md exactly
- **ZERO TOLERANCE**: No deviations from contract specifications allowed
- **CONTRACT VERIFICATION**: Must confirm understanding before implementation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AZidan/archflow](https://github.com/AZidan/archflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
