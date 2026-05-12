---
trigger: always_on
description: > **English** | [中文](#中文版本)
---

# Project Guidance for burn-in-cceverywhere-codex

> **English** | [中文](#中文版本)

This document provides project-specific guidance for working with the Codex autonomous agent system.

## Available Skills

This project provides 7 skills installed in `~/.codex/skills/`:

1. **ralph** - PRD to prd.json conversion for autonomous agent loops
2. **coding-standards** - Universal coding standards and best practices
3. **tdd-workflow** - Test-driven development workflow enforcement
4. **backend-patterns** - Backend architecture patterns and API design
5. **frontend-patterns** - React/Next.js patterns and frontend best practices
6. **security-checklist** - Security vulnerability detection and OWASP Top 10
7. **code-review-checklist** - Code review checklist for quality and maintainability

Trigger these skills by asking relevant questions (e.g., "Show me TypeScript coding standards" triggers `coding-standards`).

## Git Workflow

### Commit Message Format

```
<type>: <description>

<optional body>
```

**Types**: feat, fix, refactor, docs, test, chore, perf, ci

### Pull Request Workflow

When creating PRs:
1. Analyze full commit history (not just latest commit)
2. Use `git diff [base-branch]...HEAD` to see all changes
3. Draft comprehensive PR summary
4. Include test plan with TODOs
5. Push with `-u` flag if new branch

### Feature Implementation Workflow

1. **Plan First**
   - Create detailed implementation plan
   - Identify dependencies and risks
   - Break down into phases

2. **TDD Approach**
   - Write tests first (RED)
   - Implement to pass tests (GREEN)
   - Refactor (IMPROVE)
   - Verify 80%+ coverage

3. **Code Review**
   - Use `code-review-checklist` skill
   - Address CRITICAL and HIGH issues
   - Fix MEDIUM issues when possible

4. **Commit & Push**
   - Detailed commit messages
   - Follow conventional commits format

## Common Patterns

### API Response Format

```typescript
interface ApiResponse<T> {
  success: boolean
  data?: T
  error?: string
  meta?: {
    total: number
    page: number
    limit: number
  }
}
```

### Custom Hooks Pattern

```typescript
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value)

  useEffect(() => {
    const handler = setTimeout(() => setDebouncedValue(value), delay)
    return () => clearTimeout(handler)
  }, [value, delay])

  return debouncedValue
}
```

### Repository Pattern

```typescript
interface Repository<T> {
  findAll(filters?: Filters): Promise<T[]>
  findById(id: string): Promise<T | null>
  create(data: CreateDto): Promise<T>
  update(id: string, data: UpdateDto): Promise<T>
  delete(id: string): Promise<void>
}
```

## Performance Guidelines

### Model Selection

**gpt-5.2** (Primary model for all tasks):
- Main development work
- Code generation and pair programming
- Complex coding tasks
- Architectural decisions
- Research and analysis tasks

Note: Codex CLI uses gpt-5.2 as the unified model for all operations.

### Context Window Management

Avoid last 20% of context window for:
- Large-scale refactoring
- Feature implementation spanning multiple files
- Debugging complex interactions

Lower context sensitivity tasks:
- Single-file edits
- Independent utility creation
- Documentation updates
- Simple bug fixes

### Deep Reasoning

For complex tasks:
1. Use detailed planning before implementation
2. Break down complex problems into smaller steps
3. Verify assumptions before proceeding
4. Test thoroughly after implementation

## Build Troubleshooting

If build fails:
1. Analyze error messages carefully
2. Fix incrementally
3. Verify after each fix
4. Run full test suite before marking complete

## Multi-agents (Experimental)

This project supports Codex multi-agents for parallel task execution:

```javascript
// Example: Parallel code review
const security = spawn_agent("Security review of auth.ts", "worker")
const performance = spawn_agent("Performance review of queries.ts", "worker")

wait([security.agent_id, performance.agent_id])

close_agent(security.agent_id)
close_agent(performance.agent_id)
```

Enable in `~/.codex/config.toml`:
```toml
[features]
collab = true
```

## Testing Requirements

- Minimum 80% test coverage
- Unit tests for all functions
- Integration tests for APIs
- E2E tests for critical flows

Use `tdd-workflow` skill for TDD guidance.

## Security

Use `security-checklist` skill before commits:
- No hardcoded secrets
- Input validation on all endpoints
- SQL queries parameterized
- Output escaped (XSS prevention)
- `npm audit` clean

## Code Quality

Use `code-review-checklist` skill after writing code:
- Functions < 50 lines
- Files < 800 lines
- No deep nesting (< 4 levels)
- Immutability patterns
- Proper error handling

## Resources

- Ralph Documentation: `~/.codex/ralph/AGENTS.md`
- Skills: `~/.codex/skills/*/SKILL.md`
- Examples: `docs/examples/`

---

# 中文版本

> [English](#project-guidance-for-burn-in-cceverywhere-codex) | **中文**

本文档为 Codex 自主代理系统提供项目特定的指导。

## 可用技能

本项目在 `~/.codex/skills/` 中提供了7个技能：

1. **ralph** - PRD到prd.json的转换，用于自主代理循环
2. **coding-standards** - 通用编码标准和最佳实践
3. **tdd-workflow** - 测试驱动开发工作流程
4. **backend-patterns** - 后端架构模式和API设计
5. **frontend-patterns** - React/Next.js模式和前端最佳实践
6. **security-checklist** - 安全漏洞检测和OWASP Top 10

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hellangleZ/burn-in-cceverywhere-codex](https://github.com/hellangleZ/burn-in-cceverywhere-codex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
