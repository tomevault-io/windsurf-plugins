---
trigger: always_on
description: **System:** Veloria - Code Search Engine for the WordPress Ecosystem
---

# Claude Code Agent Instructions for Veloria

**System:** Veloria - Code Search Engine for the WordPress Ecosystem
**Tech Stack:** Go 1.26.0, PostgreSQL 17, MinIO/S3, trigram search (forked google/codesearch)
**Status:** Production - High-Performance Search Service

---

## Critical Context

You are working on a **production code search engine** that indexes and searches the **entire WordPress plugin and theme ecosystem**.

**Performance is the #1 Priority:**
- Resource usage (CPU, Memory, Disk) is a critical concern
- Search operations must remain fast under concurrent load
- Memory-mapped indexes require careful lifecycle management
- Connection pools and caches must be properly sized

**Quality Requirements:**
- Idiomatic Go code following standard conventions
- Thread-safe operations (this is a highly concurrent system)
- Graceful degradation under load
- Comprehensive error handling with OpenTelemetry integration

**Never:**
- Block search operations during index updates
- Hold locks longer than necessary (especially write locks)
- Create memory leaks (unclosed indexes, forgotten goroutines)
- Skip graceful shutdown handling
- Ignore context cancellation

---

## First Steps for Any Task

### 1. Read the Documentation

**ALWAYS start by reading:**
- [docs/architecture.md](docs/architecture.md) - System design and component overview
- [docs/api.md](docs/api.md) - REST API documentation
- [docs/configuration.md](docs/configuration.md) - Environment variables
- [docs/development.md](docs/development.md) - Local setup and debugging

**For database work:**
- Review existing migrations in `migrations/` directory

**Use Task tool with Explore agent** when you need to:
- Understand how a feature works across multiple packages
- Find all usages of a type or function
- Explore the concurrency model

### 2. Use TodoWrite for Complex Tasks

For any task with **3+ steps or affecting multiple packages**, create a todo list:

```typescript
TodoWrite({
  todos: [
    {content: "Read existing index implementation", status: "in_progress", activeForm: "Reading index code"},
    {content: "Profile memory usage", status: "pending", activeForm: "Profiling memory"},
    {content: "Implement optimization", status: "pending", activeForm: "Implementing optimization"},
    {content: "Run benchmarks", status: "pending", activeForm: "Running benchmarks"},
    {content: "Write tests", status: "pending", activeForm: "Writing tests"}
  ]
})
```

### 3. Use EnterPlanMode for Performance-Critical Changes

Use EnterPlanMode when:
- Modifying index loading or search operations
- Changing concurrency patterns (mutex usage, goroutines)
- Altering database queries or connection handling
- Modifying the hot-swap index mechanism
- Adding new background tasks

**Don't use for:**
- Simple bug fixes
- Documentation updates
- Adding new API endpoints with existing patterns

---

## Recommended Skills

Use these skills proactively when working on performance-sensitive code or database changes.

Skills are defined in [.claude/skills/](.claude/skills/) and invoked with `/skill-name`.

| Skill | Purpose | When to Use |
|-------|---------|-------------|
| [/test](.claude/skills/test/SKILL.md) | Run unit tests | After any code change |
| [/check](.claude/skills/check/SKILL.md) | Pre-push quality gate (vet + lint + tests) | Before pushing code |
| [/benchmark](.claude/skills/benchmark/SKILL.md) | Run and compare Go benchmarks | Before/after performance changes |
| [/profile](.claude/skills/profile/SKILL.md) | CPU and memory profiling with pprof | Investigating high resource usage |
| [/race-check](.claude/skills/race-check/SKILL.md) | Detect data races in concurrent code | After changes to mutexes/goroutines |
| [/coverage](.claude/skills/coverage/SKILL.md) | Test coverage analysis | Finding untested code paths |
| [/migrate](.claude/skills/migrate/SKILL.md) | Create and manage database migrations | Schema changes, new tables/indexes |
| [/lint](.claude/skills/lint/SKILL.md) | Run golangci-lint | Code quality checks |
| [/security-scan](.claude/skills/security-scan/SKILL.md) | Run gosec + govulncheck | Before committing security-sensitive code |
| [/deps](.claude/skills/deps/SKILL.md) | Tidy and verify Go modules | After adding/removing imports |
| [/generate](.claude/skills/generate/SKILL.md) | Run go generate | After templ/frontend/protobuf changes |
| [/integration-test](.claude/skills/integration-test/SKILL.md) | Run integration tests | After DB/API/storage changes |
| [/reindex](.claude/skills/reindex/SKILL.md) | Queue a reindex for an extension | Testing indexing changes |

### /test

Run Go unit tests with optional package targeting.

```
/test                             # All packages
/test ./internal/repo/...         # Specific package
/test -v ./internal/manager/...   # Verbose output
```

**Use when:** After any code change to verify correctness.

---

### /check

Run the full pre-push quality gate: vet, lint, and tests with race detection.

```
/check                            # All packages
/check ./internal/repo/...        # Specific package
```

**Use when:** Before pushing code. Mirrors CI checks.

---

### /benchmark

Run Go benchmarks and compare results to detect performance regressions.

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PeterBooker/veloria](https://github.com/PeterBooker/veloria) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
