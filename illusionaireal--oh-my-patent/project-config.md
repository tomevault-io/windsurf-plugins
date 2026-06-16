---
trigger: always_on
description: **Your idea → a full patent disclosure document** — without herding 11 AI agents into a task queue.
---

# oh-my-patent

**Your idea → a full patent disclosure document** — without herding 11 AI agents into a task queue.

## Overview

oh-my-patent is a patent disclosure automation toolkit that orchestrates 11 specialized AI agents to transform a technical idea into a complete patent disclosure document. It handles the entire lifecycle: prior art search, brainstorming, patentability assessment, drafting, review, and figure generation.

## Project Structure

```
oh-my-patent/
├── src/
│   ├── agents/           # 13 agent definitions (.md + routing)
│   │   ├── archimedes.md           # Primary orchestrator
│   │   ├── patent-landscape-analyst.md
│   │   ├── patent-innovation-architect.md
│   │   ├── patentability-evaluator.md
│   │   └── ...
│   ├── commands/         # CLI command implementations
│   │   ├── path-commands.ts        # Brainstorm path management
│   │   ├── adapt-commands.ts       # Adapter generation
│   │   └── diagram-commands.ts     # Figure rendering
│   ├── core/             # Core engine modules
│   │   ├── brainstorm-path.ts      # Decision path tracking
│   │   ├── state-manager.ts        # Workflow state machine
│   │   ├── diagram-renderer.ts     # Mermaid/PlantUML rendering
│   │   └── workflow.ts             # Stage orchestration
│   ├── adapters/         # Tool adapters (Claude Code, Codex)
│   │   ├── claude-code-adapter.ts
│   │   └── codex-adapter.ts
│   ├── skills/           # Reusable skills
│   │   ├── jurisdiction/           # Patent jurisdiction rules
│   │   ├── quality-gate/           # Threshold validation
│   │   └── prior-art-search/       # MCP-based search
│   └── tui/              # Terminal UI (Ink + React)
├── tests/
│   ├── unit/             # Unit tests (12 files)
│   ├── integration/      # Integration tests (5 files, includes CLI)
│   └── e2e/              # End-to-end tests (plugin loading)
├── docs/
│   └── superpowers/
│       └── specs/        # PRD and design specs
├── plugin.jsonc          # Plugin metadata (agents/commands/skills)
├── README.md             # English documentation
├── README.zh-CN.md       # Chinese documentation
├── CONTRIBUTING.md       # Contribution guidelines
└── LICENSE               # MIT License
```

## Key Modules

### Core Engine

- **brainstorm-path.ts**: Decision path tracking with DAG structure
  - Supports rollback, branching, and innovation history
  - Stores in `.brainstorm/` directory

- **state-manager.ts**: Workflow state machine
  - 10-stage pipeline: INIT → RESEARCH → BRAINSTORM → DRAFT → QA → FINALIZE
  - Atomic persistence to `.patent/state.json`

- **diagram-renderer.ts**: Automatic figure generation
  - Mermaid and PlantUML dual-engine support
  - Auto-inserts figures into MAIN.md

- **workflow.ts**: Stage orchestration
  - Coordinates agent handoffs
  - Manages context passing between stages

### Adapters

- **Claude Code Adapter**: Generates `.claude/` configuration
  - 13 agents as sub-agents
  - 8 commands with frontmatter permissions
  - CLAUDE.md project documentation

- **Codex Adapter**: Generates `.codex/` configuration
  - 13 agents as skills
  - Commands as custom actions
  - AGENTS.md project documentation

### CLI Commands

- `path init/overview/branch/restore`: Brainstorm path management
- `adapt setup/generate/uninstall`: Tool adapter lifecycle
- `diagram render/insert`: Figure generation pipeline
- `state show/reset/export`: Workflow state inspection

## Agent System

### Primary Orchestrator

**archimedes.md** routes tasks to 11 specialist agents:

1. **patent-landscape-analyst**: Prior art search via MCP
2. **patent-innovation-architect**: Generate innovation candidates
3. **patentability-evaluator**: Assess novelty/creativity/practicality
4. **patent-brainstorm-moderator**: Facilitate multi-round ideation
5. **patent-disclosure-writer**: Draft technical disclosure
6. **patent-disclosure-reviewer**: QA review
7. **patent-technical-responder**: Address review issues
8. **patent-adversarial-examiner**: Adversarial novelty check
9. **patent-security-engineer**: Security/cryptography review
10. **patent-product-compliance-analyst**: Standards compliance
11. **patent-path-recorder**: Decision path documentation

### Routing Logic

Archimedes reads `.patent/state.json` and routes to the appropriate agent:
- Stage `INIT` → No agent (user provides topic)
- Stage `RESEARCH` → patent-landscape-analyst
- Stage `BRAINSTORM_R1` → patent-innovation-architect + patentability-evaluator
- Stage `DRAFT` → patent-disclosure-writer
- Stage `QA_LOOP` → patent-disclosure-reviewer → patent-technical-responder

## Development Workflow

### Setup

```bash
npm install
npm run build
npm test
```

### Testing

- **Unit tests**: Fast, isolated module tests
- **Integration tests**: Multi-module interactions (includes CLI tests)
- **E2E tests**: Full plugin loading and compilation

```bash
npm test                          # All tests
npm test tests/unit/             # Unit tests only
npm test tests/integration/cli-commands.test.ts  # CLI tests
```

### Code Quality

- TypeScript strict mode (zero errors)
- 87 tests (100% passing)
- Vitest for testing
- No linting errors

## File System Layout (Project Usage)

When a user runs oh-my-patent in their project:

```
my-patent-project/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [illusionaireal/oh-my-patent](https://github.com/illusionaireal/oh-my-patent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
