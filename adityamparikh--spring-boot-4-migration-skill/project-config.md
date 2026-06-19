---
trigger: always_on
description: >
---


# Spring Boot 4 Migration Skill

Migrate Spring Boot 2.7.x applications to 3.5.x, then 3.5.x to 4.x,
and stay current across 4.x minor versions, anchored to the official
Spring Boot migration guides and release notes for each leg.

## Scope: 2.7.x → 3.5.x → 4.x and 4.x Minor Versions

This skill covers three scenarios:

1. **Legacy migration (2.7.x → 3.5.x)** — the prelude. Covered in
   `references/spring-boot-2-to-3-migration.md`: Java 8/11 → 17 baseline,
   Jakarta EE namespace migration (`javax.*` → `jakarta.*`), Spring
   Security 5 → 6, Hibernate 5 → 6, observability migration to Micrometer
   Tracing/Observation, OpenRewrite recipe automation for every step.
   Reach Boot 3.5 latest patch here, then chain into the next scenario.
2. **Major migration (3.5.x → 4.0)** — the bulk of this skill. All 9 phases,
   the gradual upgrade strategy, and the bridge system below.
3. **Minor version upgrades (4.0 → 4.1, 4.1 → 4.2, etc.)** — tracked in
   `references/minor-version-changes.md`. Minor versions may deprecate
   APIs, remove compatibility bridges, change defaults, and introduce new
   features. Check that file before bumping to any new 4.x minor version.

If your project is on Boot 2.7.x or earlier, **start with §
"Coming from Spring Boot 2.7?" below**, complete the 2 → 3 leg, then
return here for the 3 → 4 phases.

## Verify APIs Against Current Documentation

Many APIs change across these migrations. When unsure whether an API,
property, or pattern is still valid for the target version, look it up
rather than relying on model knowledge:

- Use Context7 (`mcp__claude_ai_Context7__resolve-library-id` then
  `mcp__claude_ai_Context7__query-docs`) for library docs.
- Use `WebSearch` / `WebFetch` for breaking-change announcements, CVEs,
  and community migration reports.
- Use `gh` for release notes (e.g.,
  `gh api repos/spring-projects/spring-boot/releases/latest`).

## Toolchain Version Check (Do This First)

Before starting any migration, detect the project's Java, Kotlin, Maven,
and Gradle versions. If any are below the minimums for the target Boot
version, upgrade them BEFORE bumping Boot.

Read `references/toolchain-versions.md` for the minimum/recommended
versions table and per-tool upgrade commands.

## Coming from Spring Boot 2.7? Start here.

If your project is on Spring Boot 2.7.x (or earlier), do the 2.7 → 3.5
leg FIRST. The two scenarios chain:

```
Boot 2.7.x  →  Boot 3.5.x latest  →  Boot 4.x
              (this section's prelude)   (the 9 phases below)
```

Read `references/spring-boot-2-to-3-migration.md` for the complete
2.7 → 3.5 walkthrough. It covers:

- **Toolchain bump**: Java 8/11 → 17 (re-run the 2.7 build on Java 17
  BEFORE bumping Boot, so JDK incompats surface separately from Spring
  upgrade churn)
- **Pre-flight on 2.7.18 latest patch**: clear all deprecation warnings
  and add `spring-boot-properties-migrator` for runtime diagnostics
- **OpenRewrite automation**: one-shot via
  `org.openrewrite.java.spring.boot3.UpgradeSpringBoot_3_5` (composes
  every 3.0 → 3.5 step, including Jakarta migration), or step-by-step
  per minor for large codebases
- **Jakarta EE namespace migration** (`javax.*` → `jakarta.*`)
- **Spring Security 5 → 6** (`WebSecurityConfigurerAdapter` removed,
  lambda DSL required, `requestMatchers` over `antMatchers`)
- **Hibernate 5 → 6** (`org.hibernate.orm` group ID, ID generator
  changes, naming strategy, query handling)
- **Per-minor highlights**: 3.0, 3.1, 3.2, 3.3, 3.4, 3.5
- **Property migration** via Properties Migrator + OpenRewrite
- **Observability migration** to Micrometer Tracing / Observation API

When the 2 → 3 leg is verified (build passes on Boot 3.5 latest patch
and Properties Migrator prints no warnings), **return to the
Prerequisites section below** and proceed with Phases 1–9 for 3.5 → 4.0.

## Prerequisites

### For 3.x → 4.0 migration:
- Toolchain versions meet minimums (see Toolchain Version Check above)
- Source project compiles and tests pass on Spring Boot 3.5.x (latest patch)
- Java 17+ is available (Java 21+ recommended, Java 25 supported)
- All deprecated API calls from Boot 3.x are resolved where possible
- If on Boot 3.4 or earlier, first upgrade to 3.5.x before proceeding

### For 4.x → 4.y minor version upgrade:
- Project is on the latest patch of the current minor version (e.g., 4.0.x latest)
- Review `references/minor-version-changes.md` for the target version
- Check the official release notes for the target version
- Resolve any deprecation warnings from the current version

## Choose Your Migration Strategy

**Strategy 1 — Gradual Upgrade (Recommended for enterprise/large codebases)**
Read `references/gradual-upgrade-strategy.md` FIRST. This models migration
as a dependency graph: a Day-1 baseline using compatibility bridges, then
6 independent tracks (Starters, Jackson 3, Properties, Security, Testing,
Framework 7) completed at your own pace. Key bridges:
- `spring-boot-starter-classic` — restores 3.x monolithic auto-configuration
- `spring-boot-jackson2` — keeps Jackson 2 code working alongside Boot 4
- `spring-security-access` — bridges legacy AccessDecisionManager/Voter
Use this when: multiple teams, many services, phased rollouts, or when
complete Jackson 3 / Security 7 migration will take more than one sprint.

**Strategy 2 — All-at-Once (below)**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adityamparikh/spring-boot-4-migration-skill](https://github.com/adityamparikh/spring-boot-4-migration-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
