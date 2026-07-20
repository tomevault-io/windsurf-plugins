---
trigger: always_on
description: Follow **Uncle Bob** (Clean Code, SOLID), **Kent Beck** (TDD, simple design), **Martin Fowler** (refactoring, enterprise patterns), and **Lean/Agile** (MVP, iterate, deliver incrementally).
---

# CLAUDE.md

## Guiding Principles

Follow **Uncle Bob** (Clean Code, SOLID), **Kent Beck** (TDD, simple design), **Martin Fowler** (refactoring, enterprise patterns), and **Lean/Agile** (MVP, iterate, deliver incrementally).

- **"Make the change easy (warning: this is hard), then make the easy change."** — Refactor before significant changes.
- **"As tests get more specific, the code gets more general."**
- **Four Rules of Simple Design**: passes tests → reveals intention → no duplication → fewest elements.
- **Refactor rigorously** — never create backwards-compatible shims, wrappers, or deprecation layers. Change the code directly and update all callers.
- **Split into shippable milestones**: first refactor (no behavior change) → ship. Then make the behavior change → ship. Never mix refactoring and behavior changes in the same step.
- **KISS / YAGNI** — simplest solution that works. Don't build for hypothetical future requirements: "duplication is far cheaper than the wrong abstraction."
- **Boy Scout Rule** — always leave code cleaner than you found it.
- **DORA four key metrics** — optimize for deployment frequency, lead time for changes, change failure rate, and time to restore.
- **Decision-first, not integration-first** — when adding external API integrations, start from "which business decision does this unblock?" not "which endpoints can we call?" Only add a new operation when a failing test proves the current data cannot answer a required decision.
- **Radical candor** — care personally, challenge directly. Say what you actually think: push back on bad ideas, name risks, disagree with reasoning. Don't hedge to be polite or agree to avoid friction. Ruinous empathy (soft to spare feelings) and manipulative insincerity (vague to avoid conflict) both waste the user's time.
- **Interview before planning** — ask follow-up questions before producing a plan. Surface ambiguity, missing constraints, hidden assumptions, and success criteria up front. A plan built on guessed requirements is worse than no plan. If the request is non-trivial, ask first; only skip the interview when the task is genuinely unambiguous.

## TDD-First Planning

**Structure plans as TDD steps — not production code changes.**

1. **Failing integration test** — what it asserts and why it fails
2. **For each unit of work**: failing unit test → minimal production code → green → refactor
3. **Integration test turns green**
4. **Full test suite — no regressions**

❌ "1. Add migration 2. Add enum 3. Add method 4. Write tests"
✅ "1. Failing integration test 2. Failing unit test for A → implement A → refactor 3. Integration test passes 4. Full suite green"

**Bug fixes**: don't just reproduce the specific bug — ask what tests are missing that would catch this and similar bugs.

**External integrations**: the failing integration test should assert a *business outcome* (e.g., "active board members for registry code X"), not infrastructure plumbing (e.g., "XSD generates classes"). Infrastructure setup is a prerequisite, not a test target.

## Commands

### Development
- `./gradlew bootRun` (port 9000, requires PostgreSQL)
- `./gradlew bootRun --args='--spring.profiles.active=dev,mock'` (mock EpisService)
- `docker compose up database -d` (local PostgreSQL)
- `./gradlew build` | `./gradlew build -x test`
- Spring profile `dev` for local development; do NOT use `--no-daemon`

### Testing
- `./gradlew test` (H2) | `SPRING_PROFILES_ACTIVE=pg,test ./gradlew test` (Testcontainers PostgreSQL, requires Docker)
- `./gradlew test --tests MyTest` (integration tests live under `src/test/groovy/` alongside unit tests and run as part of `test` — there's no separate `integrationTest` task)
- `./gradlew jacocoTestReport`
- Some tests require PostgreSQL and are skipped on H2

### Code Quality
- `./gradlew spotlessApply` (Google Java style via Spotless)
- `./gradlew spotlessCheck`

### Git
- Always `git add` new files immediately
- NEVER commit or push without user approval
- **NEVER commit PII — this repo is open-source.** No real names, personal/ID codes, emails, phone numbers, addresses, account/väärtpaberikonto numbers, or other personally identifiable data in code, comments, tests, fixtures, migrations, or commit messages. Use synthetic values (e.g. `38888888888`) or stable opaque IDs (payment id, `external_id`, party UUID) instead. This also applies to logs and anything pushed to an external service.

## Architecture

Spring Boot application following DDD and Spring Modulith best practices. Java 25 with preview features. Controllers are thin routing layers — extract complex logic into `@Component` classes (`*Verifier`, `*Mapper`, `*Validator`).

**Spring Modulith** — each top-level package under `ee.tuleva.onboarding` is a module. Only types in the package root are public API; sub-packages are internal. Modules communicate via Spring application events, not direct cross-module injection. Use `@ApplicationModuleTest` to verify module boundaries.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TulevaEE/onboarding-service](https://github.com/TulevaEE/onboarding-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
