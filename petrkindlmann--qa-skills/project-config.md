---
trigger: always_on
description: How AI agents should discover, load, and use skills from this repository.
---

# Agent Behavior Specification

How AI agents should discover, load, and use skills from this repository.

## Skill Discovery

On first skill use per session:

1. Check for `.agents/qa-project-context.md` in the user's project root
   - If exists: read it and use as context for all subsequent skill interactions
   - If not: suggest the user creates one using the `qa-project-context` skill
2. Read the activated skill's `SKILL.md` from `skills/<skill-name>/SKILL.md`
3. Follow the skill's Discovery Questions, skip any already answered by qa-project-context
4. Load files from `references/` only when deeper detail is needed — do not read all reference files upfront

## Cross-Skill References

- Skills reference each other with: "For [topic], see `skill-name`"
- Workflow steps that span skills include direct references at the relevant step
- The `qa-project-context` skill is the universal dependency — every skill checks for it first

## Description Quality

Each skill's YAML frontmatter `description` must be specific enough to match the right skill without fuzzy boundaries. Descriptions include what the skill does, when to use it, trigger phrases, and cross-references to related skills.

## Picking between overlapping skills

When two skills could plausibly match a request, use these disambiguation rules. The skill descriptions also carry explicit "Not for: X — use Y" anti-triggers; treat those as authoritative.

- **Strategy vs planning vs risk:** Use `risk-based-testing` first to produce the risk matrix. Use `test-strategy` for multi-quarter QA direction. Use `test-planning` for a single sprint or release.
- **Bootstrapping vs onboarding:** Use `qa-start` when starting QA on a brand-new project (no QA exists yet). Use `qa-project-bootstrap` when onboarding a QA engineer to an existing codebase.
- **AI cluster:** Use `ai-test-generation` when AI writes test code for you. Use `ai-system-testing` when AI/LLM features in your product are the thing being tested (and for LLM-layer attacks: indirect prompt injection, defend-the-tester, OWASP LLM Top 10). Use `ai-qa-review` to review existing test code. Use `ai-bug-triage` to classify CI failures. Use `bug-reproduction` to turn a defect report into a verified minimal repro and a failing regression test (it executes; `ai-bug-triage` only classifies). Use `agentic-browser-testing` for goal-driven E2E where a browser agent explores without a script (scripted Playwright stays in `playwright-automation`).
- **Production trio:** Use `testing-in-production` for safe rollout techniques (flags, canary) **during** release. Use `synthetic-monitoring` for scheduled probes that run **after** release. Use `observability-driven-testing` when prod telemetry is the **input** to designing new tests.
- **Selector maintenance:** Use `test-reliability` for runtime per-test healing when one test goes flaky. Use `selector-drift-recovery` for offline bulk regeneration after a UI refactor or redesign breaks many selectors.
- **Test-case vs test-suite work:** Use `test-case-management` to author/maintain MANUAL cases in TestRail/Xray/Zephyr/Qase (not test code — that's `ai-test-generation`). Use `test-suite-curation` to prune/restructure a whole regression suite ("should this test exist"); use `ai-qa-review` to judge whether an individual test is well-written ("is this test good").
- **Specialized-flow testing:** Use `email-testing` for inbox-capture flows (signup/reset/OTP). Use `payment-testing` for PSP-sandbox checkout/3DS/webhooks. Use `analytics-tracking-testing` to verify GA4/dataLayer/pixel data is CORRECT (whether tracking is ALLOWED under consent law is `compliance-testing`). All three defer generic endpoint contracts to `api-testing`.
- **Last resort:** Use `qa-do` ONLY when the request doesn't match any other skill's trigger phrases. If the request clearly matches another skill, invoke that skill directly.

## Tools Integration

- Skills reference tools listed in `tools/REGISTRY.md`
- Tool-specific integration guides live in `tools/integrations/`
- Examples: `playwright-automation` references the Playwright integration guide, `qa-metrics` references Allure/Grafana dashboards

## Available Skills — 50 skills across 10 categories

### Foundation
| `qa-project-context` | "set up QA context," "configure testing," first use of any skill |
| `qa-start` | "set up QA on a new project," "QA from scratch," "no QA exists yet," "/qa-start" |
| `qa-do` | "which skill should I use," "where do I start," "/qa-do" — last-resort router only |

### Strategy
| `test-strategy` | "test strategy," "multi-quarter QA direction," "QA roadmap" |
| `test-planning` | "sprint test plan," "release test plan," "what to test this sprint" |
| `risk-based-testing` | "risk matrix," "risk heatmap," "where to focus testing" — run BEFORE strategy/planning |
| `exploratory-testing` | "exploratory testing," "SBTM," "manual testing," "bug hunting" |

### Automation
| `playwright-automation` | "Playwright," "write E2E test," "page object," "new Playwright suite" |
| `cypress-automation` | "Cypress," "cy.," "component test," "Cypress Cloud" |
| `selector-drift-recovery` | "UI refactor broke tests," "redesign broke tests," "bulk update selectors," "selector drift" |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [petrkindlmann/qa-skills](https://github.com/petrkindlmann/qa-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
