---
trigger: always_on
description: Global rules and protocols for all AI interactions
---


<core_rules>

<!-- CRITICAL: Highest priority - START placement for primacy -->

- [MANDATORY] Read this file in FULL before processing ANY request
- [MANDATORY] Use tool `discord_ask` before finalizing ANY significant action
- [MANDATORY] You MUST use tool `discord_ask` before finalizing ANY session, or deciding that no further action is needed — this is non-negotiable and must be followed without exception, sessions can only be closed or be decided that all work is done with explicit user confirmation via `discord_ask` - this is critical for ensuring user alignment and preventing premature session closure
- [MANDATORY] Use subagents for specialized tasks — never do specialized work directly your the orchestrater that delegates tasks to subagents with specific expertise
- [CRITICAL] Subagents must report progress and results back to the main agent for integration and finalization and never use `discord_ask`. The main agent is responsible for all user communication and final output.
- [CRITICAL] Quality over speed — rushed or incomplete outputs are unacceptable
- [CRITICAL] `discord_embed` summary → `discord_ask` for next steps
  </core_rules>

YOU MUST DO ALL COMMUNICATION BY calling DISCORD MCP User cannot be reached in any other way, any session MUST NEVER BE concluded before calling discord_ask to verify if user has any other needs. Check your available tool usage and system your allowed to utilise it all for maxium performence. YOU MUST FOLLOW THESE RULES AT ALL COSTS! ITS EXTREMELY CRITICAL THESE PROTOCOLS ARE FOLLOWED!

1. You MUST produce high quality easy to read code
2. You MUST keep a highly structured codebase
3. You MUST use 7-parallel-Task method for efficiency
4. Its CRITICAL all communication goes through discord mcp.
5. YOU MUST USE SUBAGENTS TO ENHANCE PRODUCTIVITY

# Related Configuration

| Config File                                                   | Purpose                                        |
| ------------------------------------------------------------- | ---------------------------------------------- |
| AGENTS.md                                                     | Agent persona, available skills/agents/prompts |
| .github/instructions/styling.instructions.md                  | Dark mode patterns, color palette              |
| .github/instructions/git.instructions.md                      | Branch strategy, commits, pre-commit           |
| .github/instructions/implementation.instructions.md           | Feature workflow, parallel tasks               |
| .github/instructions/nextjs.instructions.md                   | Next.js 16 App Router best practices           |
| .github/instructions/typescript-5-es2022.instructions.md      | TypeScript 5.x coding standards                |
| .github/instructions/a11y.instructions.md                     | WCAG 2.2 accessibility guidelines              |
| .github/instructions/security-and-owasp.instructions.md       | OWASP Top 10 secure coding                     |
| .github/instructions/performance-optimization.instructions.md | Frontend/backend performance                   |
| .github/instructions/playwright-typescript.instructions.md    | E2E test writing standards                     |

## [MANDATORY] Main Agent Rule

- All/ANY Communication MUST go throrugh Discord communication tools, specifically:

1. `discord_embed` — completed work summaries, progress updates
2. `discord_ask` — confirmation before finalizing any significant action

- NEVER use `discord_ask` in subagents — all user communication must be handled by the main agent to ensure consistency and proper session management
- All significant actions, session closures, or decisions that no further action is needed MUST be confirmed with the user via `discord_ask` to ensure alignment and prevent premature closure or missed steps
- The main agent is responsible for all user communication and final output, subagents should report progress and results back to the main agent for integration and finalization without direct user communication
- You MUST assume user can not see your output in terminal so you MUST use `discord_embed` to summarize completed work, progress updates, and next steps, and use `discord_ask` to confirm before finalizing any significant action, session closure, or decision that no further action is needed — this is critical for ensuring user alignment and preventing premature session closure or missed steps

---

# Universal Protocols

## Terminal Management

- Each terminal has ONE purpose (dev server, build, tests)
- Monitor ALL terminals — check background output regularly
- Don't run multiple commands in parallel
- NEVER send a new command while previous is running — wait, close or use a new terminal

### Parallel Feature Implementation Workflow

1. **Component**: Create main component file
2. **Styles**: Create component styles/CSS
3. **Tests**: Create test files
4. **Types**: Create type definitions
5. **Hooks**: Create custom hooks/utilities
6. **Integration**: Update routing, imports, exports
7. **Remaining**: Update package.json, documentation, configuration files
8. **Review and Validation**: Coordinate integration, run tests, verify build, check for conflicts

### Context Optimization Rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zebbern/kimmmmy-discount](https://github.com/zebbern/kimmmmy-discount) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
