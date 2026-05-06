---
trigger: always_on
description: Hybrid MCP + Agent Skills implementation for spec-driven development.
---

# AI Agent Spec-Driven Development

Hybrid MCP + Agent Skills implementation for spec-driven development.

## Two Development Paths

Choose the appropriate path based on task complexity:

### Path A: Simple Task (`/simple-task`)
For small features, bug fixes, quick enhancements.

```
User: "Add a logout button"
→ /simple-task
→ References: tdd-guideline.md, principles.md, linus-review.md, owasp-top10-check.md
→ Implements with best practices
```

### Path B: Full SDD Workflow
For complex features requiring formal specification.

```
User: "Build user authentication system"
→ sdd-init → /sdd-requirements → /sdd-design → /sdd-tasks → /sdd-implement
→ Full approval workflow at each phase
```

---

## Reference Documents (Steering)

These documents provide guidance and can be referenced by skills:

| Document | Purpose |
|----------|---------|
| `tdd-guideline.md` | TDD Red-Green-Refactor methodology |
| `principles.md` | SOLID, DRY, KISS, YAGNI, Separation of Concerns |
| `linus-review.md` | Code quality and "good taste" criteria |
| `owasp-top10-check.md` | OWASP Top 10 security checklist |

Located in `.spec/steering/` (or `.kiro/steering/` for legacy projects).

---

## Architecture (v3.0)

This project uses a **comprehensive plugin architecture** with 6 component types:

- **MCP Tools**: Action-oriented operations (init, status, approve, quality-check, validate, spec-impl)
- **Skills**: Workflow guidance (requirements, design, tasks, steering, implement, commit)
- **Steering**: Project-wide conventions (TDD, principles, security, code review)
- **Rules**: Always-active guidelines (coding-style, testing, security, git-workflow)
- **Contexts**: Mode-specific prompts (dev, review, planning, security-audit, research)
- **Agents**: Specialized AI personas (planner, architect, reviewer, implementer)
- **Hooks**: Event-driven automation (pre-tool-use, post-tool-use, session events)

### Install All Components

```bash
# Recommended: Install everything
npx sdd-mcp-server install --all

# Or install specific components
npx sdd-mcp-server install --skills --steering
npx sdd-mcp-server install --rules --agents
```

## Project Context

### Paths
- Steering: `.spec/steering/`
- Specs: `.spec/specs/`
- Skills: `.claude/skills/`

### Steering Documents

**Reference Documents** (guidance for skills):
- `tdd-guideline.md`: TDD methodology and test pyramid
- `principles.md`: SOLID, DRY, KISS, YAGNI, Modularity
- `linus-review.md`: Code quality and "good taste" criteria
- `owasp-top10-check.md`: OWASP Top 10 security checklist

**Project-Specific** (describe YOUR project):
- `product.md`: Product context and business objectives
- `tech.md`: Technology stack and decisions
- `structure.md`: File organization and patterns

## Workflow

### Phase 0: Setup (One-time)
```bash
# Install all SDD components to your project
npx sdd-mcp-server install --all
```

### Phase 1: Steering (Optional)
`/sdd-steering` (Skill) - Create/update project-specific steering documents
`/sdd-steering-custom` (Skill) - Create custom steering for specialized contexts

### Phase 2: Specification Creation
1. `sdd-init` (MCP Tool) - Initialize spec with project description
2. `/sdd-requirements <feature>` (Skill) - Generate EARS-formatted requirements
3. `sdd-approve --phase requirements` (MCP Tool) - Approve requirements
4. `/sdd-design <feature>` (Skill) - Create architecture design
5. `sdd-validate-design` (MCP Tool) - GO/NO-GO design review
6. `sdd-approve --phase design` (MCP Tool) - Approve design
7. `/sdd-tasks <feature>` (Skill) - Generate TDD task breakdown
8. `sdd-approve --phase tasks` (MCP Tool) - Approve tasks

### Phase 3: Implementation
- `/sdd-implement <feature>` (Skill) - Implementation guidelines with SOLID, security, TDD
- `sdd-spec-impl` (MCP Tool) - Execute tasks with TDD methodology
- `sdd-quality-check` (MCP Tool) - Linus-style code review

### Phase 4: Commit
- `/sdd-commit` (Skill) - Commit message and PR guidelines

### Progress Tracking
- `sdd-status` (MCP Tool) - Check current progress and phases
- `sdd-context-load` (MCP Tool) - Restore project context

## MCP Tools

| Tool | Description |
|------|-------------|
| `sdd-init` | Initialize new SDD project |
| `sdd-status` | Check workflow progress |
| `sdd-approve` | Approve workflow phases |
| `sdd-quality-check` | Code quality analysis |
| `sdd-context-load` | Load project context |
| `sdd-validate-design` | Design quality validation |
| `sdd-validate-gap` | Implementation gap analysis |
| `sdd-spec-impl` | Execute tasks with TDD |
| `sdd-list-skills` | List available Agent Skills |

## Agent Skills

| Skill | Description |
|-------|-------------|
| `/simple-task` | Quick implementation with best practices (references steering docs) |
| `/sdd-requirements` | EARS-formatted requirements with quality checklist |
| `/sdd-design` | Architecture design with Linus principles |
| `/sdd-tasks` | TDD task breakdown with test pyramid guidance |
| `/sdd-implement` | Implementation guidelines (SOLID, security, TDD) |
| `/sdd-steering` | Project-specific steering documents |
| `/sdd-steering-custom` | Custom steering with inclusion modes |
| `/sdd-commit` | Commit/PR guidelines with conventional commits |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yi-john-huang/sdd-mcp](https://github.com/yi-john-huang/sdd-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
