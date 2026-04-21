---
trigger: always_on
description: | Change Type | Must Run | Can Skip |
---

# sd0x-dev-flow — Harness Engineering for Claude Code

## Required Checks (Stop Hook enforced)

| Change Type | Must Run | Can Skip |
|-------------|----------|----------|
| code files | `/codex-review-fast` -> `/precommit` | - |
| `.md` docs | `/codex-review-doc` | `/codex-review-fast` |
| Comments only | - | All |

Before PR: `/pr-review`

## Workflow

```
Feature: develop -> write tests -> /verify -> /codex-review-fast + /codex-test-review -> /precommit -> /pr-review
Bug fix: /issue-analyze -> /bug-fix -> investigate -> fix -> regression test -> /verify -> /codex-review-fast -> /precommit
```

### Auto-Loop Rule ⚠️

After editing code or docs, you **MUST** run the review command **in the same reply** — do not stop, do not ask, do not just summarize.

| After editing... | Immediately run | Then on pass |
|------------------|----------------|--------------|
| code files | `/codex-review-fast` | `/precommit` |
| `.md` docs | `/codex-review-doc` | (done) |
| Review found issues | Fix all → re-run same review | — |

**Declaring ≠ Executing**: Saying "should run review" without invoking the Skill tool is a violation.
**Summary ≠ Completion**: Outputting a table then stopping is a violation.

Full spec: @rules/auto-loop.md

## Test Requirements

| Change Type | Required Tests | File Mapping |
|-------------|---------------|--------------|
| New script/skill | `test/` required | `scripts/xxx.sh` -> `test/scripts/xxx.test.js` |
| Modify existing logic | Existing pass + new logic | `skills/<name>/SKILL.md` -> `test/skills/<name>.test.js` |
| Bug fix | Regression test | - |

Coverage: happy path + error handling + edge cases (null, empty, extremes)

## Command Quick Reference

| Command | Description | When |
|---------|-------------|------|
| `/codex-brainstorm` | Adversarial brainstorm | Exploration |
| `/req-analyze` | Requirements analysis + 1-requirements.md | Planning |
| `/feasibility-study` | Feasibility analysis | Requirements |
| `/tech-spec` | Generate tech spec | Design |
| `/review-spec` | Review tech spec | Design |
| `/deep-analyze` | Deep analysis + roadmap | Design |
| `/architecture` | Architecture design + 3-architecture.md | Design |
| `/project-brief` | PM/CTO executive summary | Design |
| `/fp-brief` | First-principles briefing | Understanding |
| `/tech-brief` | Technical briefing for developer sharing | Understanding |
| `/recap-doc` | Post-development recap document generator | Understanding |
| `/recap-ask` | Recap-bounded Q&A follow-up | Understanding |
| `/post-dev-recap` | Guided post-dev recap (scope + doc + Q&A) | Understanding |
| `/codex-architect` | Architecture advice | Design |
| `/codex-implement` | Codex writes code | Development |
| `/bug-fix` | Bug fix workflow | Bug fixing |
| `/debug` | Interactive debugging | Debugging |
| `/feature-dev` | Feature development | Development |
| `/feature-verify` | Feature verification (READ-ONLY) | Development |
| `/load-pr-review` | Load PR review comments into session | Development |
| `/pr-comment` | Post review comments to PR | Development |
| `/ask` | Context-aware Q&A with auto context gathering | Understanding |
| `/deep-explore` | Multi-wave parallel code exploration | Understanding |
| `/deep-research` | Universal multi-source research orchestration | Understanding |
| `/code-explore` | Code exploration | Understanding |
| `/code-investigate` | Dual-perspective code investigation | Understanding |
| `/git-investigate` | Track code history | Finding source |
| `/issue-analyze` | Issue deep analysis | Root cause |
| `/repo-intake` | One-time project scan | Onboarding |
| `/next-step` | Change-aware next step advisor | Development |
| `/remind` | Lightweight model correction with rule loading | Development |
| `/risk-assess` | Uncommitted code risk assessment | Development |
| `/test-deep` | Context-aware test orchestration | Development |
| `/verify` | Run tests | Development |
| `/codex-review-fast` | Quick review (diff) | **Required** |
| `/codex-review` | Full review (lint+build) | Important PR |
| `/codex-review-branch` | Full branch review | Important PR |
| `/codex-cli-review` | CLI review (full disk) | Deep review |
| `/codex-review-doc` | Review .md files | Doc changes |
| `/seek-verdict` | Independent finding verification (dismiss/confirm/clarify) | Review |
| `/codex-explain` | Explain complex code | Understanding |
| `/precommit` | lint + typecheck + test | **Required** |
| `/precommit-fast` | lint + test (no build) | Quick check |
| `/codex-security` | OWASP Top 10 | Security-sensitive |
| `/codex-test-gen` | Generate unit tests | Adding tests |
| `/codex-test-review` | Review test coverage | **Required** |
| `/post-dev-test` | Post-dev test completion | After feature |
| `/check-coverage` | Test coverage analysis | Quality |
| `/test-health` | Holistic test coverage measurement | Quality |
| `/pre-pr-audit` | Pre-PR confidence audit (5-dimension scoring) | Quality |
| `/project-audit` | Project health audit with scoring | Quality |
| `/best-practices` | Industry best practices conformance audit | Quality |
| `/necessity-audit` | Detect over-engineering in lifecycle specs (6-dim + Codex debate) | Quality |
| `/dep-audit` | Dependency vulnerability audit | Periodic / PR |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sd0xdev/sd0x-dev-flow](https://github.com/sd0xdev/sd0x-dev-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
