---
trigger: always_on
description: > **Research-backed design** (Gloaguen et al., 2026, arXiv:2602.11988): This file intentionally
---

# AGENTS.md

> **Research-backed design** (Gloaguen et al., 2026, arXiv:2602.11988): This file intentionally
> contains only **minimal, non-redundant requirements**. The peer-reviewed study
> "Evaluating AGENTS.md: Are Repository-Level Context Files Helpful for Coding Agents?"
> evaluated 4 coding agents (Claude Code, Codex, Qwen Code, OpenAI) across 438 real-world
> tasks and found:
>
> - **LLM-generated context files reduced task success by 3%** and **increased cost by 20%+**
> - **Developer-written context files only improved success by 4%** while **increasing cost by 19%**
> - **Codebase overviews had zero effect** on agent's ability to find relevant files
> - **Agents strictly follow listed instructions** (uv: 1.6× when mentioned, 0.01× when not)
> - **Conclusion: include only minimal requirements.** Unnecessary instructions make tasks harder
>   and inflate costs, contradicting current agent-developer recommendations.
>
> **Reference:** Gloaguen, T., Mündler, N., Müller, M., Raychev, V., & Vechev, M. (2026).
> Evaluating AGENTS.md: Are Repository-Level Context Files Helpful for Coding Agents?
> _arXiv preprint arXiv:2602.11988v1 [cs.SE]._

---

## Testing

- **Framework:** `go test` (stdlib) + `testify` for assertions
- Unit tests live in `*_test.go` files alongside source; integration and e2e tests under `tests/integration/` and `tests/e2e/`
- All tests **must pass** before a PR is merged; CI enforces this.
- Add or update tests for every code change, even when not explicitly requested.
- Never remove a failing test; fix it or open a follow-up issue.
- Coverage threshold: **80 %** (statements). Run `go test -coverprofile=coverage.out ./... && go tool cover -func=coverage.out`.

---

## Code style

- **Language:** Go 1.22+
- Tabs for indentation, enforced by `gofmt` / `goimports`
- Explicit error returns; no `panic` in library code
- Descriptive names over comments — `fetchUserByID` beats `getUser` + a comment

```go
// ✅ Good
func fetchUserByID(id string) (*User, error) {
	if id == "" {
		return nil, errors.New("user ID is required")
	}
	return db.FindUser(id)
}

// ❌ Bad — vague name, swallowed error, empty interface
func getUser(id interface{}) interface{} {
	u, _ := db.FindUser(fmt.Sprint(id))
	return u
}
```

---

## Git workflow

- Branch naming: `feat/<slug>`, `fix/<slug>`, `chore/<slug>`, `docs/<slug>`
- PR titles follow [Conventional Commits](https://www.conventionalcommits.org/):
  `feat:`, `fix:`, `chore:`, `docs:`, `test:`, `refactor:`
- Squash-merge into `main`; keep a clean linear history
- **Never force-push to `main`**

---

## Boundaries

| ✅ Always                                                                           | ⚠️ Ask first                                                                                       | 🚫 Never                                |
| ----------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- | --------------------------------------- |
| Write to `internal/`, `cmd/`, `tests/`, `docs/`, `specs/`                           | Add a new Go module dependency                                                                     | Commit `.env` or any secret             |
| Search peer-reviewed articles of recognized engineers or official docs for guidance | Search YouTube or publisher-owned, scholarly research databases and digital libraries for guidance | search wikipedia                        |
| Run `go test ./...` before marking a task done                                      | Modify CI/CD workflows                                                                             | Edit `vendor/` or build outputs         |
| Follow naming conventions above                                                     | Refactor across many files at once                                                                 | Remove or skip failing tests            |
| Run `gofmt -w ./...` after edits                                                    | Change the database schema                                                                         | Modify `go.sum` by hand                 |
| Code with solid reasons, facts, evidences, or researches                            | Ask before doing if you are unsure                                                                 | Guess                                   |
| block edits to this file without human authorization                                | ask the human before editing this file                                                             | edit this file without asking the human |

---

## Research findings: What to include (and exclude) in AGENTS.md

**Per the peer-reviewed study (Gloaguen et al., 2026):**

### ✅ Include (minimal, specific)

- **Tool requirements**: "Use `go` module commands (not dep or glide)" — agents follow these (1.6× usage when mentioned)
- **Build/test commands**: Exact commands agents should run, nothing more
- **Code style rules**: Specific conventions (e.g., "2-space indent", "single quotes, no semicolons")
- **Critical security**: Only OWASP Top 10 or project-specific security boundaries


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lfarizav/hdd-gsd2-hybrid-framework](https://github.com/lfarizav/hdd-gsd2-hybrid-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
