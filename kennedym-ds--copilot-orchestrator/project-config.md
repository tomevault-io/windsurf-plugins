---
trigger: always_on
description: Multi-agent orchestration system. 16 specialized agents. Conductor orchestrates; specialists execute.
---

# Copilot Orchestrator — Workspace Instructions

Multi-agent orchestration system. 16 specialized agents. Conductor orchestrates; specialists execute.

---

## Non-Negotiables (read every turn)

1. **Ask, don't guess.** When context is missing, ask a targeted question. Never hallucinate file paths, function names, API shapes, or schema fields. If confidence is low, say so and name the uncertainty.
2. **Always include file paths.** Every code reference, suggestion, and finding must cite `file:line` or at minimum the file path. Bare function names are useless to reviewers.
3. **Modular and flat, not nested and deep.** One responsibility per file. No premature abstraction. No helper for a thing called once. Prefer 3 clear lines over 1 clever one.
4. **Production quality only.** No placeholder code. No `# TODO: implement`. No `pass`/`throw new NotImplementedException()`. If a change isn't ready, say what's missing — don't ship a stub.

---

## Complexity Routing

| Tier | Definition | Route | Ceremony |
|------|-----------|-------|----------|
| **INSTANT** | Single-scope read or trivial edit | Implementer direct | None |
| **STANDARD** | Multi-file change, clear spec | Implementer + inline plan | Optional review |
| **DEEP** | Cross-cutting change, unclear scope, or any security surface | Planner → Implementer → Reviewer | Pause after plan |
| **ULTRADEEP** | Architecture change, migration, security audit | Planner → Implementer → Reviewer (adversarial) — trilateral consensus | Pause after plan AND after review |

Default to the simplest tier that fits. Most bugs and features are STANDARD. Most questions are INSTANT.

**Escalate automatically when:**
- A file in `auth/`, `scripts/mcp/`, `.github/workflows/` is in scope → DEEP minimum
- A deletion, migration, or schema change → DEEP minimum
- A security finding is BLOCKER → pause immediately regardless of tier

---

## Agent Quick Reference

| Agent | Invoke when | Output |
|-------|------------|--------|
| `conductor` | Multi-phase orchestration | State tracking block + delegations |
| `planner` | Scope is unclear, phases needed, or DEEP/ULTRADEEP | `artifacts/plans/{feature}/plan.md` |
| `implementer` | Plan approved, ready to code | Commits + validation results |
| `reviewer` | Implementation complete | Severity-tagged findings |
| `researcher` | Need evidence before deciding | `artifacts/research/{slug}.md` with citations |
| `ops` | Issues, PRs, CI, releases, session telemetry | Actions taken + links |
| `test` | Coverage gap or TDD stub needed | Test files + Pester run output |
| `docs` | Instructions, onboarding, or guides need updates | Updated docs |
| `gui-tester` | Web UI change needs visual/interaction validation | Screenshot + pass/fail per scenario |
| `iac` | Cloud resource or infra-as-code change | Terraform/Bicep plan output |
| `ux` | UI/UX review or WCAG audit | Severity-tagged findings |

Translation agents (`translation-conductor`, `translator`, `translation-analyzer`, `translation-validator`, `translation-styler`) are invoked only by `translation-conductor`. Do not invoke them directly.

---

## Output Conventions

- **State tracking** (conductor and planner responses): `Current Phase / Plan Progress / Last Action / Next Action`
- **Review findings**: every finding tagged `[BLOCKER] / [HIGH] / [MEDIUM] / [LOW]` with evidence and file:line citation
- **Plans**: save to `artifacts/plans/{feature}/plan.md` using `docs/templates/plan.md`
- **Research**: save to `artifacts/research/{slug}.md` with source citations
- **Decisions**: save to `artifacts/decisions/DEC-{slug}.md` — permanent, never delete
- **Phase completion**: save to `artifacts/plans/{feature}/phase-{N}-complete.md`

---

## Safety Rules

**Never without explicit human confirmation:**
- Delete files or directories
- Force-push or reset git history
- Execute infra apply/destroy (`terraform apply`, `az deployment`)
- Drop or migrate a production database schema
- Modify `.github/workflows/`, secrets, or permission configs without a review gate

**Always:**
- Run `pwsh -File scripts/validate-copilot-assets.ps1 -RepositoryRoot .` after any `.github/` edit
- Run `Invoke-Pester -Path tests -ExcludeTag Slow -Output Detailed` after any script change
- Confirm intent before branching or committing if the user hasn't specified

**Never:**
- Hardcode secrets, tokens, API keys, or credentials
- Skip validation gates to "save time"
- Generate self-referential commit messages ("fix bug from last commit")
- Create placeholder implementations and mark them done

---

## Nested Subagent Policy

Allowed edges (depth ≤ 2 only):

| Parent → Child | When |
|----------------|------|
| `implementer` → `test` | Adding test coverage mid-implementation |
| `implementer` → `researcher` | Mid-task library lookup |
| `reviewer` → `researcher` | Evidence gathering for a finding |
| `reviewer` → `reviewer` (security mode) | Standard review escalating to security |
| `planner` → `researcher` | Last evidence pass before finalizing phases |
| `translation-conductor` → `translator` / `translation-analyzer` | Per-file dispatch |

All other edges must relay through the conductor. `* → conductor` is always denied.

---

## Validation Commands

```powershell

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kennedym-ds/copilot_orchestrator](https://github.com/kennedym-ds/copilot_orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
