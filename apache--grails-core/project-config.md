---
trigger: always_on
description: SPDX-License-Identifier: Apache-2.0
---

<!--
SPDX-License-Identifier: Apache-2.0

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

# Agent Guide for grails-core

> **IMPORTANT**: This is the Grails Framework source repository (60+ modules), NOT a Grails application.
> For building Grails apps, see `.agents/skills/grails-developer/SKILL.md`.

## Quick Reference

```bash
# Build (no tests)
./gradlew build -PskipTests

# Build single module
./gradlew :grails-core:build

# Run tests
./gradlew :<module>:test
./gradlew :<module>:test --tests "com.example.SomeSpec"

# Style check
./gradlew codeStyle

# Out of memory? Set:
export GRADLE_OPTS="-Xms2G -Xmx5G"
```

## Critical Rules

1. **Use `jakarta.*` NOT `javax.*`** - All packages migrated to Jakarta EE 10
2. **Use `@GrailsCompileStatic`** - Not plain `@CompileStatic` in Grails artefact classes
3. **Use `GrailsWebRequest.lookup()`** - For thread-safe request context in tests
4. **No wildcard imports** - Use explicit imports
5. **4 spaces, no tabs** - See `.editorconfig`
6. **Apache license header** - Required on all new source files
7. **New features require docs** - Any user-facing change must include or update documentation in `grails-doc`; do not merge features without corresponding doc coverage
8. **No internal APIs in docs** - Only document public APIs; never reference internal or package-private classes and methods in user-facing documentation
9. **Test via public APIs** - Tests must exercise behavior through the same APIs an end user calls; never invoke internal implementations, package-private methods, or bypass the public surface directly
10. **Always review and extend tests** - Review existing unit and functional tests before making changes; every code change must include new or enhanced tests that cover the affected behavior
11. **Every code touch must update all tests for the changed class** - When a class is modified, find and update every test that covers it — unit, integration, and TCK. Do not leave any existing test out of sync with the new code.
12. **Clean violations before commit** - Before every automated commit, run `./gradlew clean aggregateViolations :grails-test-report:check --continue` from the root and ensure that `build/reports/violations/CHECKSTYLE_VIOLATIONS.md`, `build/reports/violations/CODENARC_VIOLATIONS.md`, `build/reports/violations/PMD_VIOLATIONS.md`, and `build/reports/violations/SPOTBUGS_VIOLATIONS.md` report no issues. Also review the test result reports under `grails-test-report/build/reports/tests/` and ensure there are no failures. The aggregate reports are wired as test finalizers and will be attempted after failures, but `--continue` is required for comprehensive full-suite reports.
13. **Mandatory test coverage** - Any class touched in a commit MUST be covered with tests that verify all behavior. You must run ALL tests in the affected module(s) and ensure they pass before committing.
14. **The BOM must manage the latest version** - `validateDependencyVersions` enforces that the BOM (`dependencies.gradle`) manages a version `>=` every transitively-resolved version. When it fails, **bump the version in `dependencies.gradle`** so the BOM wins — never silence it with `allowedBomOverrides` or an exclusion unless there is an explicit, documented conflict or an agreed-upon workaround. See [Dependency Management](#dependency-management).

## Available Skills

> **AI AGENTS - MANDATORY**: Before writing or modifying any code, you **MUST** read the relevant skill file(s) below. Do not write Groovy/Grails code without first loading these instructions:
> - Writing Grails code → Read `.agents/skills/grails-developer/SKILL.md`
> - Writing Groovy code → Read `.agents/skills/groovy-developer/SKILL.md`
> - Writing Java code → Read `.agents/skills/java-developer/SKILL.md`
> - Upgrading applications to Grails 8 → Read `.agents/skills/grails-8-upgrade/SKILL.md`
> - Writing Hibernate code → Read `.agents/skills/hibernate-developer/SKILL.md`
> - Fixing style/analysis violations → Read `.agents/skills/violation-fixer/SKILL.md`
> - Fixing broken test → Read `.agents/skills/test-fixer/SKILL.md`
> - Indexing code -> Read `.agents/skills/codebase-memory/SKILL.md`
>
> Use your file reading capability to load the skill content before proceeding with any code changes.

| Skill | Path | Use For |
|-------|------|---------|
| **grails-developer** | `.agents/skills/grails-developer/SKILL.md` | Current Grails apps, GORM, controllers, views |
| **groovy-developer** | `.agents/skills/groovy-developer/SKILL.md` | Groovy 5 syntax, closures, DSLs, Spock |
| **grails-8-upgrade** | `.agents/skills/grails-8-upgrade/SKILL.md` | Upgrading Grails applications from 7.x to 8 |
| **java-developer** | `.agents/skills/java-developer/SKILL.md` | Java 21 features, Groovy interop |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/grails-core](https://github.com/apache/grails-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
