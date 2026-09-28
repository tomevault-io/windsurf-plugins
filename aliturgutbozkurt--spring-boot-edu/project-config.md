---
trigger: always_on
description: A bilingual (Turkish + English), module-by-module Spring Boot course.
---

# CLAUDE.md — Spring Boot Education Project

A bilingual (Turkish + English), module-by-module Spring Boot course.
Every module ships **runnable example code, tests, lesson docs (MD + PDF, TR + EN), exercises and solutions**.
The source of truth for scope is [SPEC.md](SPEC.md); the build order is [tasks/plan.md](tasks/plan.md); the open work is [tasks/todo.md](tasks/todo.md).

## Workflow (Spec-Driven Development, agent-skills plugin)

1. **Spec first.** Never add a module, dependency or feature that is not in `SPEC.md`. If scope changes, update `SPEC.md` first, then `tasks/plan.md`, then `tasks/todo.md`.
2. **One task at a time** from `tasks/todo.md` (`/build`). Tick the checkbox only after its *Verify* step passes.
3. **Test-first for code** (`/test`): write the failing test, make it pass, refactor.
4. **Review before a module is "done"** (`/review`): run the Module Definition of Done below.
5. Keep changes small: one task ≈ one commit (Conventional Commits: `feat(06-data-jpa-postgres): ...`, `docs(06-data-jpa-postgres): ...`).
6. **GitHub issues:** every task/module has an issue (numbers in `tasks/todo.md`, map in `tasks/issues.json`, milestones = phases). Commits say `Refs #N`; the commit finishing a task says `Closes #N`. Module issues close only when all a/b/c items are done.

## Tech Stack (pinned — do not change without asking)

| Item | Version |
|---|---|
| Java | **27** (`maven.compiler.release=27`, no `--enable-preview` in lesson code unless the lesson is *about* a preview feature) |
| Spring Boot | **4.1.1** (Spring Framework 7.0.x, Spring Security 7.1.x, Spring Data 2026.0.x) |
| Build | Maven via wrapper (`./mvnw`), multi-module |
| Test | JUnit 6, AssertJ, Mockito, Testcontainers 2.x with `@ServiceConnection` |
| Infra (Docker) | PostgreSQL, MongoDB, Elasticsearch, Redis, Kafka (KRaft), Hazelcast, Ollama, Grafana LGTM |
| Extra BOMs | Spring Cloud 2025.1.3, Spring AI 2.0.1, Spring Modulith 2.1.1 (Spring gRPC 1.1.1 is in the Boot BOM) |
| Kubernetes | kind (local), Kustomize, Helm (capstone) |
| Docs | Markdown → PDF via Pandoc + XeLaTeX (runs in Docker, nothing to install locally) |

Library versions come from the Spring Boot BOM. Never hard-code a version that the BOM already manages.

## Commands

```bash
# JDK: Maven must run on JDK 27 (the machine default Maven JDK may be older)
export JAVA_HOME=$(/usr/libexec/java_home -v 27)

./mvnw -q verify                               # build + test everything (lessons + solutions)
./mvnw -pl modules/06-data-jpa-postgres/lesson -am verify   # one module
./mvnw -pl modules/06-data-jpa-postgres/lesson spring-boot:run   # run a lesson (starts its Docker services automatically)
./mvnw -Pexercises -pl modules/06-data-jpa-postgres/exercise test # student exercise tests (fail until solved)

docker compose --profile postgres up -d        # start infra manually (profiles: postgres, mongo, elastic, redis, kafka, hazelcast, observability, all)
docker compose down -v                         # stop + wipe volumes

./scripts/build-pdfs.sh                        # all MD → PDF (Dockerized Pandoc)
./scripts/build-pdfs.sh 06-data-jpa-postgres            # one module
./scripts/check-module.sh 06-data-jpa-postgres          # structure, TR/EN parity, snippets, fresh PDFs
./scripts/check-module.sh --strict 06-data-jpa-postgres # + finished content — required for Definition of Done
./scripts/sync-snippets.sh 06-data-jpa-postgres         # refresh doc code blocks from // tag:: regions in the source
./scripts/new-module.sh 06-data-jpa-postgres --title-tr "..." --title-en "..." --infra postgres   # scaffold (id must be in SPEC)
./scripts/kind-up.sh / kind-down.sh            # local Kubernetes cluster (modules 22, 23, capstone)
```

## Repository Layout

```
pom.xml                         # root aggregator + shared plugin config
build-parent/pom.xml            # parent for all modules (Boot parent, Java 27, enforcer, test config)
compose.yaml                    # all infra services, grouped by Docker Compose profiles
docs/templates/                 # lesson/exercise templates (tr + en)
scripts/                        # build-pdfs.sh, check-module.sh, new-module.sh (logic in scripts/lib/coursetool.py)
modules/NN-slug/
  README.md                     # bilingual index: what, how to run, links to docs
  (no compose.yaml by default)   # lesson/ reuses the root compose.yaml via spring.docker.compose.profiles.active
  lesson/                       # Maven module: runnable examples + tests (always green)
  exercise/                     # Maven module: starter code with TODOs + tests (red until solved; only in -Pexercises)
  solution/                     # Maven module: reference solution; same tests as exercise/ (always green)
  docs/tr/ders.md  docs/tr/odevler.md  docs/tr/ders.pdf  docs/tr/odevler.pdf
  docs/en/lesson.md docs/en/exercises.md docs/en/lesson.pdf docs/en/exercises.pdf
capstone/                       # final project combining all technologies
tasks/plan.md, tasks/todo.md    # plan and task list
```

## Code Conventions

- groupId `com.springbootedu`; base package `com.springbootedu.<moduleslug>` (e.g. `com.springbootedu.datajpa`). Example sub-packages by *feature*, not by layer: `book/`, `order/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aliturgutbozkurt/spring-boot-edu](https://github.com/aliturgutbozkurt/spring-boot-edu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
