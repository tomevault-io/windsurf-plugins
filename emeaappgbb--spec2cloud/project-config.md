---
trigger: always_on
description: This project uses the **spec2cloud** framework — a spec-driven development process
---

# Copilot Instructions

## Project Overview
This project uses the **spec2cloud** framework — a spec-driven development process
where specifications (PRD → FRD → Gherkin) are the source of truth, and implementation
is driven by automated tests generated from those specifications.

## Framework
- Orchestration: See `AGENTS.md` for the master orchestrator instructions
- Skills: See `.github/skills/*/SKILL.md` ([agentskills.io](https://agentskills.io/specification) standard)
- Ecosystem: Search [skills.sh](https://skills.sh/) for community skills via `npx skills find [query]`
- State: See `.spec2cloud/state.json` for current project state
- Audit: See `.spec2cloud/audit.log` for execution history

## Coding Conventions
- Write minimal code to pass tests — no gold-plating
- Follow existing patterns in the codebase
- All code must be covered by tests (unit, integration, or e2e). **Exception:** Brownfield Track B features use manual verification checklists when automated tests are not feasible (see AGENTS.md Testability Gate).
- No hardcoded secrets — use environment variables
- No hardcoded URLs — use configuration
- Error handling: every external call must have error handling
- Logging: use structured logging (not console.log/print)
- Comments: only when code intent is non-obvious

## Spec-Driven Development Rules
- **Never implement features not in the specs** (specs/frd-*.md)
- **Never modify tests** without human approval — tests are the contract
- **Never skip tests** — no `test.skip()`, no `xit()`, no `@pending`, no commenting out. Tests are proof that specs are met.
- **Always check Gherkin scenarios** before implementing a feature
- **Always run tests** after making changes — the FULL suite, not just "relevant" tests
- If a spec seems wrong, flag it — do not silently deviate

## Test Discipline Gospel
Tests are the **proof** that specifications have been implemented correctly. They are not optional, not skippable, and not negotiable. See AGENTS.md §9 for the full protocol.

Key rules:
- **Tests = proof of spec completion.** A feature without passing tests is not done.
- **Phase order is sacred.** Tests → Contracts → Implementation → Verify. Never skip or reorder.
- **All tests must pass before advancing.** No deployment, no commit to main while any test fails.
- **Regression is mandatory.** After every change, run ALL tests — not just new ones.

### When the User Spots a Problem
At any point during the flow, if the user identifies something wrong:
1. **Pause** current work
2. **Offer** to generate a test that captures the problem
3. **Present** the test for user approval (mandatory human gate)
4. **Fix** minimally after approval
5. **Verify** the test passes and full regression is green
6. **Resume** the interrupted work

This is the **bug-spot protocol** — it applies in every phase. The user's approval of the test is mandatory because the test defines what "fixed" means.

## File Organization
```
specs/          → Specifications (PRD, FRDs, Gherkin)
specs/contracts/→ Contracts (API specs per feature, infrastructure resources)
specs/ui/       → UI/UX design artifacts (screen-map, design-system, component-inventory, prototypes/, walkthroughs)
specs/features/ → Gherkin feature files (greenfield + brownfield Track A)
specs/adrs/     → Architecture Decision Records
specs/docs/     → Brownfield extraction outputs (technology/, architecture/, testing/)
specs/assessment/ → Assessment outputs (modernization, security, cloud-native, etc.)
specs/increment-plan.md → Ordered increment delivery plan
specs/tech-stack.md → Resolved tech stack (all technology decisions, wiring, deployment)
e2e/            → Playwright end-to-end tests (integration slice)
tests/          → Cucumber.js BDD tests (integration slice)
src/shared/     → Contract types shared between API and Web
src/api/        → Express.js backend (API slice)
src/web/        → Next.js frontend (Web slice)
infra/          → Azure infrastructure (Bicep)
.spec2cloud/    → Framework state and config
```

## Git Conventions
- Commit messages: `[impl] {feature}/{slice} — slice green` (e.g., `[impl] user-auth/api — slice green`)
- Branch naming: `spec2cloud/{phase}/{feature}` (e.g., `spec2cloud/impl/user-auth`)
- Always commit `.spec2cloud/state.json` and `.spec2cloud/audit.log` with changes
- Never commit secrets, .env files, or node_modules

## Testing Hierarchy
1. **Unit tests** — API slice: fastest feedback, run on every backend change
2. **Component tests** — Web slice: build verification and component tests
3. **Playwright e2e** — Full user flow tests, run against Aspire environment
4. **Gherkin step definitions** — BDD behavioral tests, run against Aspire environment
5. **Smoke tests** — post-deployment verification (per increment)

> **All Cucumber and Playwright tests run against the Aspire environment** (`aspire start` + `aspire wait`), which orchestrates API and Web services identically to production.
> **Iterative delivery:** Each increment goes through the full test → implement → deploy cycle. After each increment, ALL tests pass and the app is deployed.

## Human Gates
The agent MUST pause and ask for human approval at these points:
- After FRD review (before UI/UX design)
- After UI/UX design (before increment planning)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EmeaAppGbb/spec2cloud](https://github.com/EmeaAppGbb/spec2cloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
