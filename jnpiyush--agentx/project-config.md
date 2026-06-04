---
trigger: always_on
description: AI Agent Guidelines - map of all resources, quick-reference rules, and pointers to detailed docs.
---


# AI Agent Guidelines

> **Single source of truth for repository workflow guidance.**

> **Map to all AgentX resources.** For workflow details, see [docs/WORKFLOW.md](docs/WORKFLOW.md).
> For agent role definitions, see individual files in `.github/agents/`.

---

## Retrieval-Led Reasoning

**IMPORTANT**: Prefer retrieval-led reasoning over pre-training-led reasoning for ALL implementation tasks.
Always `read_file` the relevant SKILL.md, instruction file, or spec before generating code.
Do NOT rely on training data for project-specific patterns, conventions, or APIs.
If a skill, spec, or doc exists in the workspace, read it first; generate second.

---

## Quick Reference

### Issue-First Rule

Every piece of work SHOULD start with an issue. See [docs/WORKFLOW.md](docs/WORKFLOW.md) for full flow.

```bash
# GitHub Mode
gh issue create --title "[Story] Add /health" --label "type:story"  # Creates #42
git commit -m "feat: add health endpoint (#42)"

# Local Mode (issues optional by default)
git commit -m "feat: add user login"
```

Toggle enforcement: `.agentx/agentx.ps1 config set enforceIssues true`

### Classification

| Type | Label | Route To |
|------|-------|----------|
| Broken? | `type:bug` | Engineer |
| Research? | `type:spike` | Architect |
| Docs only? | `type:docs` | Engineer |
| Pipeline/deploy? | `type:devops` | DevOps Engineer |
| ML/AI/eval? | `type:data-science` | Data Scientist |
| Testing/cert? | `type:testing` | Tester |
| Power BI? | `type:powerbi` | Power BI Analyst |
| Large/vague? | `type:epic` | Product Manager |
| Single capability? | `type:feature` | Architect |
| Otherwise | `type:story` | Engineer |

### Commit Format

```
type: description (#issue-number)
```

Types: `feat`, `fix`, `docs`, `test`, `refactor`, `perf`, `chore`

For final delivery in GitHub mode, plain `(#123)` is traceability only. Use `fixes #123`, `closes #123`, or `resolves #123` in the final PR body or delivery commit so GitHub closes the issue automatically.

### Security Checklist

- [PASS] No hardcoded secrets
- [PASS] SQL parameterization (NEVER concatenate)
- [PASS] Input validation on all endpoints
- [PASS] Dependencies scanned
- Blocked commands: `rm -rf /`, `git reset --hard`, `drop database`

### Local Files First Rule

All agents MUST create deliverable files locally using `editFiles` -- MUST NOT use `mcp_github_create_or_update_file` or `mcp_github_push_files` to push files directly to GitHub. Users must be able to review files locally before committing.

### Quality Loop Hard Rule

> HARD RULE: Every agent MUST run `.agentx/agentx.ps1 loop start -p "<task description>"` as the ABSOLUTE FIRST action before any file edit or tool call. Minimum 5 iterations means at least 5 loop passes before completion is allowed; the loop is NOT done until `.agentx/agentx.ps1 loop complete -s "<summary>"` succeeds. No exceptions. The pre-commit hook blocks review artifacts when no completed loop exists.

### Compound Engineering Hard Rule

> HARD RULE: Every agent MUST resolve Compound Capture before declaring work Done. After delivery and review are complete, classify the capture decision:
> - **Mandatory**: Work produces reusable workflow, architecture, review, or operator guidance -> create `docs/artifacts/learnings/LEARNING-<issue>.md`
> - **Optional**: Narrow or low-leverage work -> capture is helpful but not required
> - **Skip**: Trivial, transient, or duplicated -> record skip rationale in the issue close comment
>
> Work is NOT Done until Compound Capture is resolved. The pre-commit hook validates LEARNING file structure when staged. See [docs/WORKFLOW.md](docs/WORKFLOW.md) for the full Compound Capture contract.

### Pipeline Phase Compliance Hard Rule

> HARD RULE: Every agent MUST follow their prescribed pipeline phases IN SEQUENCE. No phase may be skipped. Each phase has a completion gate -- the gate MUST pass before advancing to the next phase. Agents MUST NOT write deliverables before completing research phases, MUST NOT implement before planning, MUST NOT approve before verifying all checks.
>
> See the Role Pipeline Reference table (below the Agents table) for each role's phases and key delivery gate. The pre-commit hook validates deliverable structure for key artifacts (PRD, ADR, UX). Use `.agentx/agentx.ps1 workflow <agent>` to print the phase list for any role.

### CLI Quick Reference

```powershell
.\.agentx\agentx.ps1 loop start -p "Task description"  # FIRST command - start before any work
.\.agentx\agentx.ps1 loop iterate -s "Progress summary"  # After each verification pass
.\.agentx\agentx.ps1 loop complete -s "All gates passed"  # LAST command - required before handoff
.\.agentx\agentx.ps1 ready                    # Show unblocked work
.\.agentx\agentx.ps1 state -a engineer -s working -i 42
.\.agentx\agentx.ps1 deps 42                  # Check blockers
.\.agentx\agentx.ps1 workflow engineer        # Show workflow steps
.\.agentx\agentx.ps1 loop status                # Check quality loop status
.\.agentx\agentx.ps1 config show               # View configuration
```

---

## Agents (21 total)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jnPiyush/AgentX](https://github.com/jnPiyush/AgentX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
