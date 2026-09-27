---
trigger: always_on
description: `JAVA_HOME` must be JDK 21. The system JDK is newer, so Gradle fails without it.
---

# JobHunter - Project AGENTS.md

## Commands

### Java environment (read first)

`JAVA_HOME` must be JDK 21. The system JDK is newer, so Gradle fails without it.

```bash
export JAVA_HOME=$(/usr/libexec/java_home -v 21)
# -> /opt/homebrew/Cellar/openjdk@21/21.0.12/libexec/openjdk.jdk/Contents/Home
```

Notes:
- `$HOME/.gradle/jdks` does **not** exist on this machine. `Makefile` and
  `scripts/dev.sh` probe it first and fall back to `/usr/libexec/java_home -v 21`,
  which is what actually resolves.
- The JDK is Homebrew `openjdk@21`, not Temurin.

### API (Spring Boot) - run from `api/`

Build file is `api/build.gradle.kts`.

```bash
export JAVA_HOME=$(/usr/libexec/java_home -v 21)

# Install/build
./gradlew build -x test

# Run dev server (Gradle)
./gradlew bootRun

# Run JAR directly (faster restart)
"$JAVA_HOME/bin/java" -jar build/libs/jobhunter-api-0.0.1-SNAPSHOT.jar --spring.liquibase.enabled=false --spring.quartz.auto-startup=false

# Build fat JAR
./gradlew bootJar

# Run ALL unit tests (excludes @Tag("integration"))
./gradlew test

# Run single test class
./gradlew test --tests "dev.jobhunter.filter.LanguageFilterImplTest"

# Run integration tests (needs Docker/Testcontainers)
./gradlew integrationTest

# Type check (compile only)
./gradlew compileJava
```

Note: `api/build.gradle.kts` filters several test sources out of `compileTestJava`
(in-flight production code — `people/**` tests, `CrawlService`, `CliStrategy`,
`linkedin/LinkedInDescriptionEnricherTest`). Those classes are silently skipped by
`./gradlew test` until the filter is removed.

### Dashboard (React) - run from `dashboard/`

```bash
npm install
npm run dev          # Vite on http://localhost:3000, proxies /api → VITE_API_URL or http://localhost:8089
npm run build        # tsc && vite build
npm run preview      # vite preview
npm test             # vitest run
npm test -- src/test/Companies.test.tsx   # single test file
npx tsc --noEmit     # type check only
```

### MCP Server (TypeScript) - run from `mcp-server/`

This is the **JobHunter MCP server** (stdio). It is not started by `make dev`.

```bash
npm install
npm run build        # tsc
npm run start        # node dist/index.js
npm run dev          # tsx src/index.ts
npm test             # vitest run
npm test -- src/test/tools.test.ts  # single test
npx tsc --noEmit     # type check
```

### Dev stack (Makefile)

```bash
make dev         # DB + LinkedIn MCP + API + Dashboard
make restart     # stop, then `dev` — does NOT rebuild the JAR (see below)
make stop        # stop API/Dashboard/MCP (DB left running)
make status      # what is running, incl. the API's current port
make logs        # tail /tmp/jobhunter/api.log
make logs-all    # tail api.log + dashboard.log + mcp.log
make build       # build-api
make build-api   # gradlew bootJar -x test
make build-dashboard
make test        # gradlew test (API)
make clean       # gradlew clean (API)
```

**`make restart` does NOT rebuild the JAR.** `scripts/dev.sh:68` only builds when
the JAR is missing (`if [[ ! -f "$API_JAR" ]]`), so a restart happily runs stale
bytecode. To pick up Java changes:

```bash
make stop && make build-api && make dev
```

Stop first — rebuilding while the JVM is running has caused JAR corruption.

### Admin Endpoints (manual triggers)

Dev-stack API listens on **8089** (`api/src/main/resources/application.yaml`).
The live port is also written to `/tmp/jobhunter-api.port` by `scripts/dev.sh`.

```bash
PORT=$(cat /tmp/jobhunter-api.port 2>/dev/null || echo 8089)

curl -X POST http://localhost:$PORT/api/admin/crawl                    # crawl all due endpoints
curl -X POST http://localhost:$PORT/api/admin/crawl/{endpointId}       # crawl single endpoint
curl -X POST http://localhost:$PORT/api/admin/score                    # re-score all unscored jobs
curl -X POST http://localhost:$PORT/api/admin/backfill-descriptions    # backfill SmartRecruiters descriptions
curl http://localhost:$PORT/api/admin/health                           # endpoint health report
```

## Project Structure

```
jobhunter/
├── api/                         # Spring Boot 3.3.5 backend (Java 21, build.gradle.kts)
│   └── src/main/java/dev/jobhunter/
│       ├── ai/                  # AI providers - cover letters, resume tailoring
│       ├── config/              # Spring config, CORS, Quartz, WebClient, RetryFilter
│       ├── controller/          # REST controllers (/api/*), incl. AdminController
│       ├── discovery/           # Company discovery engine
│       ├── dto/                 # Response DTOs + DtoMapper (static methods, not MapStruct)
│       ├── filter/              # Job filters: Role, Location, Language (Lingua), YOE, Deduplication
│       ├── indeed/              # Indeed source integration
│       ├── ingestion/           # Aggregator ingestion pipeline, StrategyRegistry,
│       │                        #   AggregatorDescriptionEnricher, DescriptionBackfiller
│       ├── linkedin/            # LinkedIn integration
│       ├── mcp/                 # MCP client wiring (LinkedIn MCP)
│       ├── model/               # JPA entities
│       │   └── enums/           # enums, incl. AtsType
│       ├── people/              # Career-ops / outreach subsystem

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sdeonvacation/jobhunter](https://github.com/sdeonvacation/jobhunter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
