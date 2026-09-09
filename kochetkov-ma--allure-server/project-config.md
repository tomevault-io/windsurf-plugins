---
trigger: always_on
description: [DICT: SB=Spring Boot, CP=@ConfigurationProperties, !==never, ->=therefore]
---

# CLAUDE.md

[DICT: SB=Spring Boot, CP=@ConfigurationProperties, !==never, ->=therefore]

## 0. Lazy-load index

`.claude/rules/*.md` auto-load via `paths:` globs. Load other surfaces only when the task touches them.

| Surface | Load when | Key files |
|---------|-----------|-----------|
| Pinned versions | pinning ANY dep/plugin/Docker tag/frontend asset; quarterly re-audit | `.claude/convention/versions.md` -- SINGLE source of version truth; !=inline version numbers elsewhere |
| Architecture | designing/wiring modules, generation pipeline, plugin SPI, security, persistence, CI | `.claude/convention/project-architecture.md` |
| Etalon patterns | writing a new class in any layer (DTO, CP, config, SPI, Feign, repo, scheduler) | `.claude/convention/reference-patterns.md` |
| Testing | writing/fixing tests, slices, fixtures | `.claude/convention/testing-conventions.md` |
| Task board | tracking/transitioning ANY task, grooming backlog, overall status | `.claude/features/board.md` (canonical) + `.claude/features/TRACKER.md` (procedure); auto-rule `.claude/rules/tasks.md`; dump unclear items in `.claude/features/backlog/` as `*.md`; skill `.claude/skills/task-board/`. Old `.claude/tasks/` deprecated (pointer only) |
| Team / agents | delegating work, picking an owner agent | §4 + `.claude/teams/default/team.md`; definitions in `.claude/agents/` |
| Memory sync | instruction memory drifted from code (after big merges/refactors) | skill `.claude/skills/memory-sync/` -- `/memory-sync [session\|branch\|recent[:N]\|all]` |

## 1. What allure-server is

Open-source Allure report server. One SB monolith, main `ru.iopump.qa.allure.Application`, group `ru.iopump.qa`.

- REST API: `/api/result` (upload allure-results zip) + `/api/report` (generate/list/delete); Swagger UI `/swagger-ui.html`.
- Web UI: JTE templates (`src/main/jte/`) + HTMX + Alpine.js + Tailwind standalone binary (NO Node anywhere); controllers in `web/`.
- Embedded Allure 2 generator (`helper/AllureReportGenerator`) + plugin SPI `AllureServerPlugin`; external plugin JARs via `/ext` volume (`-Dloader.path=/ext`, PropertiesLauncher).
- YouTrack TMS: OpenAPI-generated Feign client (`openApiGenerate` -> `build/generated/`, post-processed in `build.gradle`).
- Auth: DB-backed (`security/DbUserDetailsService`, API-token filter, forced password change) + optional OAuth2 (`oauth` profile). H2 file DB default, Postgres via datasource env vars.
- Docker: self-contained two-stage `eclipse-temurin:25` (jdk-alpine build -> jre-alpine runtime, non-root, `EXPOSE 8080`).

Old docs claiming Vaadin / Java 21 / Gradle 8.8 / Node are STALE -- Vaadin and Node are REMOVED.

## 2. Build / run / test (verified)

Java 25 + Gradle wrapper required. Tailwind binary auto-downloaded by `tailwindDownload` task.

| Command | Purpose |
|---------|---------|
| `./gradlew build` | full: YouTrack codegen + JTE precompile + Tailwind CSS + compile + tests + bootJar |
| `./gradlew test` | all JUnit 5 tests |
| `./gradlew test --tests ru.iopump.qa.allure.helper.UtilTest` | one test class |
| `./gradlew test --tests "*.UtilTest.methodName"` | one test method |
| `./gradlew bootRun` | run locally on `:8080` |
| `./gradlew bootJar` | runnable jar -> `build/libs/allure-server-*.jar` |
| `./gradlew openApiGenerate` | regen YouTrack Feign client (auto before `compileJava`); regen, !=hand-edit `build/generated/` |
| `./gradlew tailwindBuild` | rebuild `static/css/app.css` from `src/main/frontend/input.css` (generated, gitignored) |
| `./gradlew dependencyUpdates` | dependency upgrade report (ben-manes) |
| `docker build -t allure-server .` | self-contained image -- bootJar built inside stage 1 |
| `docker compose -f docker-compose.yml up` | Postgres example; `docker-compose-h2.yml` = H2 variant |

## 3. Rules

| # | Rule |
|---|------|
| 1 | Manager-mode: non-trivial work via sub-agents (§4 roster). Trivial reads/single-file edits direct. |
| 2 | Task tracking: `.claude/features/board.md` = canonical list. Follow `.claude/features/TRACKER.md` (folder == status; update board on every transition). Dump unclear items in `.claude/features/backlog/` as `*.md`. Auto-rule: `.claude/rules/tasks.md`. |
| 3 | Task-tracker bookends (MANDATORY, every non-trivial task): FIRST step = claim/create the task on the board (`todo`->`progress`); LAST step = sync statuses + board. !=hand-edit `.claude/features/**` -- delegate to the `task-tracker` agent. |
| 4 | Pin exact versions everywhere. !=`latest`/floating ranges. Single source: `.claude/convention/versions.md` -- bump table + source file in lockstep. |
| 5 | No AI attribution in commits/PRs. |
| 6 | English only in all written artifacts: CLAUDE.md, `.claude/**/*.md`, comments, commits, PR text, docstrings, log messages. |
| 7 | Tests: GIVEN/WHEN/THEN structure, concrete assertions (`isEqualTo`/`hasSize`, !=`isNotNull` alone), no `if` in tests. Detail: `.claude/rules/test-*.md`. |
| 8 | No fabrication. Fact not in this repo -> ask. |
| 9 | Git identity: this repo is bound to GitHub account `kochetkov-ma` ONLY. !=use any `tfin` account (`mkochetkov_tfin`, `GITHUB_TFIN_*`) for commits, author identity, or push here. See §5. |

## 4. Team


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kochetkov-ma/allure-server](https://github.com/kochetkov-ma/allure-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
