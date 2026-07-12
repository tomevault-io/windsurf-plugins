---
trigger: always_on
description: This project has an **Agent Skill Framework** for Android/Kotlin, compliant with [Agent Skills Specification](https://agentskills.io).
---

# GitHub Copilot Instructions

## 🎯 Mode Activation

This project has an **Agent Skill Framework** for Android/Kotlin, compliant with [Agent Skills Specification](https://agentskills.io).

**Entry Point**: `agent-skills/SKILL.md` - Read this first for skill metadata and overview.

Rules are applied **on-demand**, not always.

### Activation Commands
Use these keywords to activate specific modes:

| Command | Mode | When to use |
|---------|------|-------------|
| `/skill` or `@gen-skill` | Skill Mode | Creating/editing Agent Skills |
| `/android` or `@gen-*` | Android Mode | Android code (ViewModel, Repository, etc.) |
| `/rules` or `/guide` | Show Rules | List all available guides & rules |
| `/help` | Help | Show available commands |
| `@review` or `/review` | Review Mode | Review code for rule compliance |
| `@review-changes` | Review Changes | Review git changes/PR for violations |

### Helper Commands (Quick Access)
When user uses these commands, load and summarize the corresponding resource:

| Command | Action | Resource |
|---------|--------|----------|
| `/skill-info` | Load & summarize | `agent-skills/SKILL.md` |
| `/context` | Load & summarize | `agent-skills/skills/AI_CONTEXT.md` |
| `/summary` | Load & summarize | `agent-skills/skills/AGENT_SUMMARY.md` |
| `/cheat` | List all | `agent-skills/*_CHEAT_SHEET.md` files |
| `/templates` | List available | `agent-skills/skills/templates/` directory |
| `/examples` | List available | `agent-skills/skills/templates/examples/` directory |
| `/decision` | Load & present | `agent-skills/skills/guides/00-decision-tree.md` |

### Pattern Quick Reference Commands
When user uses these commands, load the relevant guide section:

| Command | Topic | Primary Resource |
|---------|-------|------------------|
| `/validate` | Validation DSL | `agent-skills/VALIDATION_CHEAT_SHEET.md` + `agent-skills/skills/guides/24-validation-rules.md` |
| `/errors` | Error Handling | `agent-skills/ERROR_HANDLING_CHEAT_SHEET.md` |
| `/test` | Testing | `agent-skills/TESTING_CHEAT_SHEET.md` + `agent-skills/skills/guides/11-testing.md` |
| `/compose` | Compose | `agent-skills/COMPOSE_CHEAT_SHEET.md` + `agent-skills/skills/guides/05-jetpack-compose.md` |
| `/nav` | Navigation | `agent-skills/skills/guides/13-navigation.md` |
| `/di` | DI Setup | `agent-skills/skills/guides/08-dependency-injection.md` |
| `/flow` | State Management | `agent-skills/skills/guides/07-state-management.md` |
| `/offline` | Offline-First | `agent-skills/skills/guides/10-offline-first.md` |
| `/clean` | Clean Code | `agent-skills/skills/AGENT_SUMMARY.md` Anti-patterns section |
| `/kotlin2` | Kotlin 2.0+ | `agent-skills/skills/guides/32-modern-kotlin-features.md` |
| `/feedback` | Agent Feedback | `agent-skills/skills/guides/33-agent-feedback-loop.md` |
| `/logging` | Logging & Analytics | `agent-skills/skills/guides/34-logging-analytics.md` |
| `/gradle` | Build Optimization | `agent-skills/skills/guides/35-gradle-optimization.md` |

### Auto-Activation (when context is clear)
- Editing files in `agent-skills/skills/templates/` → Skill Mode
- Editing `*ViewModel.kt`, `*Repository.kt`, `*UseCase.kt` → Android Mode
- Using `@gen-*` prompts from AI_PROMPTS.md → Corresponding mode

### Default Behavior (No activation)
- Answer naturally and flexibly
- No forced patterns or templates
- General coding assistance

---

## 📂 File Priority Order (Read First)

**Before starting ANY work**, read context files in this order:

### Quick Start (Always Read)
1. `agent-skills/SKILL.md` - Entry point (Agent Skills spec compliant)
2. `agent-skills/skills/AI_CONTEXT.md` - Quick reference (5 min read)
3. `agent-skills/skills/AGENT_SUMMARY.md` - Core rules & patterns

### Task-Specific Files

| Task | Primary Template | Supporting Guide |
|------|-----------------|------------------|
| Skill | `agent-skills/skills/templates/examples/DomainSkillGuide.kt` | `agent-skills/skills/guides/30-skill-analysis.md` |
| ViewModel | `agent-skills/skills/templates/examples/UseCaseViewModelExample.kt` | `agent-skills/skills/guides/07-state-management.md` |
| Compose UI | `agent-skills/skills/templates/compose/ComposeExample.kt` | `agent-skills/skills/guides/05-jetpack-compose.md` |
| Repository | `agent-skills/skills/templates/examples/UserRepositoryImpl.kt` | `agent-skills/skills/guides/01-architecture.md` |
| Pipeline | `agent-skills/skills/templates/examples/PipelineExamples.kt` | `agent-skills/skills/guides/31-pipeline-patterns.md` |
| Skill Tests | `agent-skills/skills/templates/testing/SkillTestTemplate.kt` | `agent-skills/skills/guides/11-testing.md` |
| ViewModel Tests | `agent-skills/skills/templates/testing/ViewModelTestTemplate.kt` | `agent-skills/skills/guides/11-testing.md` |
| Repository Tests | `agent-skills/skills/templates/testing/RepositoryTestTemplate.kt` | `agent-skills/skills/guides/14-offline-first.md` |
| Compose Tests | `agent-skills/skills/templates/testing/ComposeScreenTestTemplate.kt` | `agent-skills/skills/guides/05-jetpack-compose.md` |
| Integration Tests | `agent-skills/skills/templates/testing/IntegrationTestTemplate.kt` | `agent-skills/skills/guides/01-architecture.md` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uselflabs/android-agent-skills](https://github.com/uselflabs/android-agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
