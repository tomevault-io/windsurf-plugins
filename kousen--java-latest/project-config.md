---
trigger: always_on
description: This repository contains comprehensive Java examples and demonstrations covering features from Java 8 through Java 25+. It serves as both a workshop resource and code reference for multiple presentation formats. The codebase emphasizes educational clarity while maintaining modern Java best practices and SonarCloud quality standards.
---

# CLAUDE.md - Java Latest Repository Context

## Repository Overview

This repository contains comprehensive Java examples and demonstrations covering features from Java 8 through Java 25+. It serves as both a workshop resource and code reference for multiple presentation formats. The codebase emphasizes educational clarity while maintaining modern Java best practices and SonarCloud quality standards.

## Recent Major Improvements (2025)

### Code Quality Modernization
The repository underwent comprehensive quality improvements to align with modern Java practices while maintaining its educational purpose:

1. **HttpClient Lifecycle Management**
   - Converted all HttpClient instances to static final fields
   - Fixed async operation lifecycle issues with Java 21+ AutoCloseable implementation
   - Ensures proper resource sharing and prevents premature client closure

2. **Exception Handling Standardization**
   - Implemented proper InterruptedException handling throughout codebase
   - Added `Thread.currentThread().interrupt()` calls in all catch blocks
   - Separated IOException and InterruptedException handling for clarity

3. **Modern Java Pattern Adoption**
   - Migrated from `collect(Collectors.toList())` to `toList()` terminal operations
   - Implemented record patterns in switch expressions where applicable
   - Enhanced pattern matching usage in data-oriented programming examples

4. **SonarCloud Integration**
   - Configured comprehensive code quality analysis with educational code standards
   - Disabled inappropriate rules for demo/educational code (commented code, System.out.println, generic exceptions)
   - Maintained quality gates while preserving educational clarity

## Recent Improvements (2026)

1. **Dependency Security via BOM Constraints**
   - Transitive CVEs (e.g., Jetty via wiremock-jetty12) are fixed with `testImplementation(platform(...))` BOM imports rather than direct dependency overrides
   - All dependency coordinates, including the security BOM pins, live in `gradle/libs.versions.toml` so the versions plugin flags stale pins
   - BOM constraints are sticky upward but become silent no-ops once transitives catch up — prune them when `dependencyUpdates` shows they're obsolete

2. **JUnit Parallel Execution**
   - Tests run in parallel, configured in `src/test/resources/junit-platform.properties`
   - Opted into JUnit's new `WORKER_THREAD_POOL` executor (predictable thread counts vs. ForkJoinPool's elastic compensation)

3. **Workshop Exercise Organization**
   - Exercises numbered sequentially 1-17 in the main flow; Java 8 refresher exercises are A1-A3 in the appendix
   - Students paste scaffolds from `workshop-exercises.md` into `src/test/java/exercises/` (see the README in that package); reference solutions live in `exercises/solutions/`
   - The `exercises` package must always contain at least one tracked file, or IDEs can't display it as a paste target (git doesn't track empty directories)

4. **Self-Contained Presentations**
   - Slide content images are hosted in `images/` and referenced with relative paths — no live network needed to present or export PDFs
   - Exception: the decorative `background:` URLs in both decks' Slidev frontmatter remain remote; a missing background degrades gracefully

## Key Architecture Decisions

### Dual Presentation Strategy
The repository supports **two distinct presentation formats**:

1. **Workshop Presentation** (`slides.md`)
   - Educational/training focus
   - Step-by-step Java evolution coverage
   - Paired with `workshop-exercises.md`
   - Designed for hands-on learning sessions

2. **Java Tier List Presentation** (`slides-java-tier-list.md`)
   - Entertainment/engagement focus  
   - Feature ranking and discussion format
   - Interactive audience participation
   - 90-minute presentation timing
   - Uses `images/` folder for tier list graphics

### Repository Consolidation Decision
**Important**: The tier list presentation was originally developed in a separate repository (`java-tier-list`) but was **migrated here** for these reasons:
- Single source of truth for Java code examples
- Easier distribution (one repo link)
- Code and slides always in sync
- Leverages existing repository popularity
- Avoids complexity of submodules or multiple repos

### Code Organization Philosophy
- **Package-based organization** - Each Java feature has its own package
- **Comprehensive examples** - Both simple demos and real-world usage
- **Test coverage** - 160+ tests demonstrating functionality
- **Presentation support** - Code examples referenced directly in slides
- **Educational clarity** - Code optimized for learning, not just production patterns

## Technical Configuration

### Build Setup
- **Java 25 LTS** with **preview features enabled**
- Gradle configuration includes `--enable-preview` for:
  - Compilation (`JavaCompile` tasks)
  - Testing (`Test` tasks)
  - Execution (`JavaExec` tasks)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kousen/java_latest](https://github.com/kousen/java_latest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
