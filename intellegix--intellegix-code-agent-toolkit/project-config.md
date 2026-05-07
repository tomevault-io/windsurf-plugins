---
trigger: always_on
description: - <Your Name> | <Role>
---

# Global CLAUDE.md v3.0 — Compact
# <Your Name> | <Your Org> | Patterns: ~/.claude/patterns/

## Identity
- <Your Name> | <Role>
- <Organization> (<Description>)
- <City, State> | <Timezone>

## Tech Stack
- Python (70%), TypeScript (25%), Kotlin (5%)
- Backend: FastAPI, Flask, Node.js | Frontend: React, Next.js, TailwindCSS
- AI/ML: Claude API, Perplexity, Ollama | DB: PostgreSQL, SQLite, Redis

## Commands
`/research`, `/smart-plan`, `/fix-issue`, `/implement`, `/implement-perplexity`, `/review`, `/handoff`, `/mcp-setup`, `/browser-test`, `/mcp-deploy`, `/council-refine`, `/export-to-council`, `/council-extract`, `/cache-perplexity-session`, `/portfolio-status`, `/frontend-e2e`, `/solve-perplexity`, `/session-audit`, `/raken-perplexity`, `/raken-api` (Defined in `~/.claude/commands/*.md`)

## Models
opus (complex arch) | sonnet (code) | haiku (quick) | sonnet+web (research)

## Date/Time Awareness (MANDATORY)
- **Hooks auto-inject**: `[TIME SYNC]` messages appear on every session start AND before every prompt via `UserPromptSubmit` hook — these are authoritative, always use the most recent one
- **Never guess dates**: If no `[TIME SYNC]` is visible in context, run `python -c "from datetime import datetime; print(datetime.now().strftime('%Y-%m-%d %H:%M:%S %A'))"` before any time-sensitive work
- **Time-sensitive operations**: Creating dated files, scheduling, commits, reports — always reference the latest `[TIME SYNC]` timestamp
- **After compaction**: SessionStart hook re-injects time automatically, but verify with a Bash command if doing date-dependent work
- **File naming**: Use ISO 8601 format (YYYY-MM-DD) for sortability in all generated filenames

## Response Style
- Be direct and technical - skip preambles
- Show code first, explain after
- State assumptions explicitly when uncertain
- Don't ask for confirmation - just do it, then summarize

## Code Standards

### Python (Primary)
- Type hints on ALL functions: `def process(data: dict) -> Result[T]:`
- Async/await for I/O: `async def fetch(url: str) -> dict:`
- Pydantic for validation, logging over print, Result pattern for errors
- See `patterns/PYTHON_PATTERNS.md`

### TypeScript
- ES modules only, explicit return types on exports
- React Query for server state, Zustand for client, Zod for validation
- See `patterns/TYPESCRIPT_PATTERNS.md`

### Naming
Py: snake_case vars/funcs, PascalCase classes, SCREAMING_SNAKE consts, snake_case.py files
TS: camelCase vars/funcs, PascalCase classes, SCREAMING_SNAKE consts, kebab-case.ts files

## Pattern References
Error handling: `patterns/PYTHON_PATTERNS.md#result-pattern`; Validation: `patterns/PYTHON_PATTERNS.md#pydantic-validation`; API: `patterns/API_PATTERNS.md`; Testing: `patterns/TESTING_PATTERNS.md`; Security: `patterns/SECURITY_CHECKLIST.md`; MCP: `patterns/MCP_PATTERNS.md`; Browser: `patterns/BROWSER_AUTOMATION_PATTERNS.md`

## Security
- Load credentials from environment: `API_KEY = os.environ["API_KEY"]` — never hardcode; use pydantic-settings or python-dotenv
- Validate ALL external input with Pydantic/Zod; parameterized queries only — never concatenate user input into SQL
- Never log passwords, tokens, PII, or API keys; audit log all mutations with user_id + timestamp
- Full checklist: `patterns/SECURITY_CHECKLIST.md`

## Git Workflow
- Branches: `feature/IGX-123-desc` | `bugfix/ASR-456-fix` | `hotfix/critical-patch`
- Commits: `feat(scope): add feature` | `fix(scope): resolve bug` | `refactor(scope): improve code`
- Pre-commit: type check (`mypy src/` or `npm run type-check`); run affected tests; no hardcoded secrets; new env vars in `.env.example`

### GitHub Merge Protocol (MANDATORY)
Protected branches (`master`, `main`) require PRs. **Never merge directly.** Always:
1. Create PR via `gh pr create`
2. Wait for CI: `gh pr checks <number>` — ALL checks must pass (green)
3. If a check fails, investigate and fix (re-run if transient, fix code if real)
4. Only after all checks pass: `gh pr merge <number> --merge`
5. Never use `--admin` to bypass failed checks unless explicitly instructed

## Agent Behavior
- Before changes: read files first, understand context; Bugs: failing test first; Features: types/interfaces first; Refactors: tests pass before AND after
- Code gen: complete working code (no TODOs), all imports, follow codebase patterns, docstrings on public funcs
- Verify after changes: type-check; run affected tests; check circular deps if new imports

### Failure Escalation via Research (MANDATORY — FIRST UNCLEAR FAILURE TRIGGERS RESEARCH)
**After the first fix attempt fails and the root cause is NOT obvious from the error message + stack trace + diff, IMMEDIATELY escalate to `/research-perplexity`.** Do not retry with variations. Do not debug in circles. Unclear failure = research mode. $0 cost, ~90s — prevents 10K+ token debugging spirals.

**Escalation ladder (execute in order):**
1. **Tier 0 — Self-check** (0s): Read the error. If the fix is mechanically obvious from the message alone (wrong arg, missing import, syntax error), fix it. No escalation needed.
2. **Tier 1 — Memory check** (5s): Search project MEMORY.md and global MEMORY.md for the error signature. If a prior research fix exists, apply it directly. No escalation needed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [intellegix/intellegix-code-agent-toolkit](https://github.com/intellegix/intellegix-code-agent-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
