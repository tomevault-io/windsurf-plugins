---
trigger: always_on
description: MultiAgent-Claude is a sophisticated orchestration framework for AI development, built with Node.js and JavaScript. This repository follows a multi-agent architecture pattern and emphasizes modularity, extensibility, and cross-platform compatibility.
---

# AGENTS.md - Repository Guidelines

## Project Overview

MultiAgent-Claude is a sophisticated orchestration framework for AI development, built with Node.js and JavaScript. This repository follows a multi-agent architecture pattern and emphasizes modularity, extensibility, and cross-platform compatibility.

**Key Technologies**: Node.js, JavaScript, Playwright, GitHub Actions, CommonMark  
**Architecture**: Multi-agent orchestration with CLI tooling  
**Development Philosophy**: Research-plan-execute pattern, memory-driven development, cross-platform compatibility

## Directory Structure

```
MultiAgent-Claude/
├── cli/                    # CLI implementation
│   ├── index.js           # Main entry point
│   └── commands/          # Command implementations
├── Examples/              # Agent and command templates
│   ├── agents/            # Agent template library
│   └── commands/          # Command templates
├── templates/             # Project setup templates
│   ├── workflows/         # CI/CD workflows
│   └── tests/            # Test templates
├── tests/                 # Test suite
├── .claude/              # Claude-specific configuration
│   ├── agents/           # Project agents
│   └── tasks/            # Session contexts
├── .chatgpt/             # OpenAI-specific configuration
│   ├── bundles/          # Optimized file bundles
│   └── roles/            # Agent role instructions
└── .ai/
    └── memory/                        # Unified persistent knowledge base (all platforms)
        ├── project.md                 # Project-wide context
        ├── patterns/                  # Successful implementation patterns
        ├── decisions/                 # Architectural Decision Records (ADRs)
        └── index.json                 # Quick lookup index
```

## Agent & Command Templates

- `Examples/commands/README.md` lists reusable CLI command templates and selection guidelines (e.g., `/wave-execute`, `/generate-tests`, `/implement-feature`).
- `Examples/agents/README.md` catalogs available orchestrator and specialist agents with their trigger keywords.

### Command Templates
- `/wave-execute` - Runs the full 7-wave orchestration cycle with context propagation
- `/generate-tests` - Generates comprehensive Playwright test plans
- `/implement-feature` - Coordinates full-stack feature delivery via planning then execution
- `TEMPLATE-COMMAND` - Starter template for creating new commands

### Orchestrator Agents
- code-review-orchestrator
- fullstack-feature-orchestrator
- infrastructure-migration-architect
- issue-triage-orchestrator
- master-orchestrator
- parallel-controller
- wave-execution-orchestrator

### Specialist Agents
- ai-agent-architect
- aws-backend-architect
- aws-deployment-specialist
- backend-api-frontend-integrator
- codebase-truth-analyzer
- cpp-plugin-api-expert
- documentation-architect
- frontend-ui-expert
- implementation-verifier
- mongodb-specialist
- multimodal-ai-specialist
- playwright-test-engineer
- playwright-visual-developer
- ui-design-auditor
- vercel-deployment-troubleshooter

Each specialist has a matching ChatGPT role specification in `.chatgpt/roles/<agent>-role.md` for OpenAI platforms.

## Role Guidelines

### When Working on CLI Development
**Triggers**: CLI, commands, terminal, command-line interface, mac command

**Approach**:
1. Review existing commands in `cli/commands/`
2. Follow Commander.js patterns for new commands
3. Add comprehensive help text and examples
4. Test with various input scenarios
5. Update CLI documentation in README.md
6. Run `npm test` to verify functionality

**Focus Areas**: User experience, error handling, cross-platform compatibility

### When Working on Agent Development
**Triggers**: Agents, orchestration, specialists, task delegation, automation

**Approach**:
1. Check agent templates in `Examples/agents/`
2. Follow research-plan-execute pattern
3. Define clear trigger keywords and patterns
4. Specify output locations in `.claude/doc/`
5. Document quality standards and examples
6. Test agent invocation and output

**Focus Areas**: Clear responsibilities, comprehensive planning, actionable outputs

### When Working on Memory System
**Triggers**: Memory, patterns, ADRs, knowledge base, documentation

**Approach**:
1. Review `.ai/memory/` structure
2. Document patterns after 2+ successful uses
3. Create ADRs for architectural decisions
4. Update project.md with conventions
5. Maintain index.json for quick lookups
6. Ensure cross-platform compatibility

**Focus Areas**: Knowledge preservation, pattern recognition, decision documentation

### When Working on Testing
**Triggers**: Tests, testing, validation, quality assurance, debugging

**Approach**:
1. Use Playwright for CLI and web testing
2. Maintain tests in `tests/` directory
3. Follow existing test patterns
4. Ensure >80% code coverage
5. Run full suite before commits
6. Update visual baselines when needed

**Focus Areas**: Edge cases, regression prevention, cross-platform validation

### When Working on Cross-Platform Integration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ancient23/MultiAgent-Claude](https://github.com/Ancient23/MultiAgent-Claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
