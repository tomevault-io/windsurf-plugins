---
trigger: always_on
description: **This project uses Apache Maven with the Maven Wrapper (`./mvnw`).**
---

## Build System: Maven

**This project uses Apache Maven with the Maven Wrapper (`./mvnw`).**

### Commands
| Task | Command |
|------|---------|
| Full build | `./mvnw install` |
| Fast build (skip checks) | `./mvnw install -Pfastinstall` |
| Compile only | `./mvnw compile` |
| Run tests | `./mvnw test` |
| Run single test | `./mvnw test -pl modules/cpr -Dtest=BroadcasterTest` |
| Skip tests | `./mvnw install -DskipTests` |
| Checkstyle only | `./mvnw checkstyle:checkstyle` |
| PMD only | `./mvnw pmd:check` |

### Module Build
Build a specific module (faster iteration):
```bash
# Build only atmosphere-runtime (cpr)
./mvnw install -pl modules/cpr -DskipTests

# Build only spring-boot-starter
./mvnw install -pl modules/spring-boot-starter -DskipTests

# Build only quarkus extension
./mvnw install -pl modules/quarkus-extension -DskipTests
```

## Git Workflow

### Hooks Setup
**Run this at the START OF EVERY SESSION:**
```bash
git config core.hooksPath .githooks
```
This enables pre-commit, commit-msg, and pre-push hooks. Sessions get archived/revived, so this must run EVERY time you start working.

**NEVER use `--no-verify` when committing or pushing.** The hooks enforce:
- Apache 2.0 copyright headers on all Java source files
- No unused or duplicate imports in staged Java files
- Commit message format (max 2 lines, conventional commits recommended)
- No AI-generated commit signatures
- Pre-push: Maven build must pass (via validation marker)

### Commit Message Format
Use conventional commit prefixes. The commit-msg hook warns if missing:
```
type(scope): description
```
Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `perf`, `ci`, `build`, `revert`

Examples:
```
feat(cpr): add WebSocket reconnection support
fix(spring-boot): resolve auto-configuration ordering
chore: update Jetty to 12.0.16
```

Rules enforced by hooks:
- Maximum 2 non-empty lines (summary + optional detail)
- First line under 100 characters (50-72 recommended)
- No AI-generated signatures (`Co-Authored-By: Claude`, `Generated with`, etc.)
- **NEVER add `Co-authored-by: Copilot` or any AI co-author trailer** — the commit-msg hook will reject it
- Do not add ANY trailer lines (Co-authored-by, Signed-off-by, etc.) to commit messages

### Branch Strategy
- Main branch: `main` (development), `atmosphere-2.6.x` (legacy)
- Feature branches for new features
- Bug fixes go directly to `main`

## Project Structure

```
atmosphere/
├── config/                        (checkstyle, PMD rulesets)
├── modules/
│   ├── cpr/                       (atmosphere-runtime - core framework)
│   ├── spring-boot-starter/       (Spring Boot 4.0 integration)
│   └── quarkus-extension/         (Quarkus 3.21+ integration)
│       ├── runtime/
│       └── deployment/
├── samples/
│   ├── chat/                      (Jetty embedded sample)
│   ├── embedded-jetty-websocket-chat/
│   ├── spring-boot-chat/
│   └── quarkus-chat/
├── atmosphere.js/                 (TypeScript client library)
├── assembly/
└── pom.xml                        (4.0.57-SNAPSHOT, JDK 21)
```

### Key Artifacts
| Module | ArtifactId | Description |
|--------|-----------|-------------|
| Core | `atmosphere-runtime` | Main framework JAR |
| Spring Boot | `atmosphere-spring-boot-starter` | Spring Boot 4.0 auto-configuration |
| Quarkus Runtime | `atmosphere-quarkus-extension` | Quarkus 3.21+ runtime |
| Quarkus Deployment | `atmosphere-quarkus-extension-deployment` | Quarkus build-time processing |

## Writing Code

- Java version: **21** (configured in root pom.xml `<release>21</release>`)
- License: Apache 2.0
- All Java files MUST have the copyright header (enforced by pre-commit hook)
- Commit without AI assistant-related commit messages
- Do not add AI-generated commit text in commit messages
- **NEVER add `Co-authored-by:` trailers to commits** — no Copilot, no Claude, no AI attribution of any kind
- NEVER USE `--no-verify` WHEN COMMITTING CODE
- Match the style and formatting of surrounding code
- Make the smallest reasonable changes to get to the desired outcome
- NEVER remove code comments unless you can prove they are actively false

### Copyright Header (Required)
All Java source files must start with:
```java
/*
 * Copyright 2008-2026 Async-IO.org
 *
 * Licensed under the Apache License, Version 2.0 (the "License"); you may not
 * use this file except in compliance with the License. You may obtain a copy of
 * the License at
 *
 * http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
 * WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
 * License for the specific language governing permissions and limitations under
 * the License.
 */
```

### Java Code Style
- Prefer composition over inheritance
- Use `try-with-resources` for all `AutoCloseable` resources
- Prefer `Optional` over null returns for public APIs
- Use `var` for local variables when the type is obvious from context
- Prefer `switch` expressions (JDK 21) over if/else chains for enum handling
- Use sealed classes/interfaces where appropriate
- Use records for immutable data carriers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Atmosphere/atmosphere](https://github.com/Atmosphere/atmosphere) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
