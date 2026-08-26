---
trigger: always_on
description: **Type**: Spring Boot Java Application
---

# OpsFinder - Project Context

## Project Overview

**Name**: OpsFinder
**Type**: Spring Boot Java Application
**Version**: 0.0.1-SNAPSHOT
**Java Version**: 21
**Spring Boot Version**: 4.0.0
**Package**: com.igsl.opsfinder

## Technology Stack

### Core Framework
- **Spring Boot**: 4.0.0
- **Java**: 21 (with toolchain configuration)
- **Build Tool**: Gradle
- **Dependency Management**: Spring Dependency Management Plugin 1.1.7

### Dependencies
- `spring-boot-starter` - Core Spring Boot functionality
- `spring-boot-starter-test` - Testing framework
- `junit-platform-launcher` - Test runtime

### Testing
- JUnit Platform for unit and integration tests
- Spring Boot Test support

## Project Structure

```
OpsFinder/
├── src/
│   ├── main/
│   │   ├── java/com/igsl/opsfinder/
│   │   │   └── OpsFinderApplication.java
│   │   └── resources/
│   │       └── application.properties
│   └── test/
│       └── java/com/igsl/opsfinder/
│           └── OpsFinderApplicationTests.java
├── docs/
│   ├── IMPLEMENTATION_PLAN.md  (main project plan)
│   └── task/                   (individual task documentation)
├── gradle/
├── build.gradle
├── settings.gradle
└── gradlew / gradlew.bat
```

## Development Workflow Rules

### CRITICAL: Task Planning & Execution Protocol

**MANDATORY WORKFLOW - FOLLOW THESE STEPS FOR EVERY TASK:**

1. **THINK FIRST**: Thoroughly analyze the problem and read all relevant codebase files
2. **WRITE PLAN**: Create a detailed plan in `docs/task/[task-name].md` with a checklist of todo items
3. **SEEK APPROVAL**: Stop and check in with the user to verify the plan before proceeding
4. **EXECUTE**: Work through todo items one by one, marking each as complete as you go
5. **COMMUNICATE**: Provide high-level explanations of changes at each step (not verbose details)
6. **SIMPLIFY**: Make every task and code change as simple as possible - minimal code impact
7. **REVIEW**: Add a review section to the plan document summarizing all changes and relevant info
8. **UPDATE MAIN PLAN**: Add completed task to `docs/IMPLEMENTATION_PLAN.md` with reference to task file

### CORE PRINCIPLES

**SIMPLICITY ABOVE ALL**:
- Every change should impact as little code as possible
- Avoid massive or complex modifications
- Only touch code directly relevant to the task
- Goal: Zero bugs introduced through minimal surface area changes

**SENIOR DEVELOPER MINDSET**:
- DO NOT BE LAZY - EVER
- NEVER USE TEMPORARY FIXES OR WORKAROUNDS
- ALWAYS FIND AND FIX THE ROOT CAUSE
- NO SHORTCUTS - PROPER SOLUTIONS ONLY
- Think deeply, execute precisely

### Task Planning Format

All plans must be written to `docs/task/[task-name].md` with this structure:

```markdown
# Task: [Task Name]

## Analysis
[Your understanding of the problem after reading the codebase]

## Todo List
- [ ] Item 1
- [ ] Item 2
- [ ] Item 3

## Review
[Summary of changes - filled in after completion]
```

### Documentation Structure

**Task Documentation**:
- Individual tasks: `docs/task/[task-name].md`
- Main project plan: `docs/IMPLEMENTATION_PLAN.md`

**After completing a task**:
1. Ensure task file is in `docs/task/` directory
2. Update `docs/IMPLEMENTATION_PLAN.md` with:
   - Brief description of completed task
   - Date completed
   - Reference to task file: `See: docs/task/[task-name].md`

**Example entry in IMPLEMENTATION_PLAN.md**:
```markdown
- ✅ **CORS environment configuration** - ALLOWED_ORIGINS now reads from environment variables (2025-12-11)
  - See: `docs/task/configure-cors-from-env.md`
```

## Development Guidelines

### Code Style & Conventions
- **Package Structure**: Follow `com.igsl.opsfinder.*` convention
- **Spring Annotations**: Use appropriate Spring stereotypes (@Service, @Repository, @Controller, etc.)
- **Java Version**: Utilize Java 21 features where appropriate
- **Naming**: Use clear, descriptive names following Java conventions

### Build & Run Commands

**Build Project**:
```bash
./gradlew build
```

**Run Application**:
```bash
./gradlew bootRun
```

**Run Tests**:
```bash
./gradlew test
```

**Clean Build**:
```bash
./gradlew clean build
```

### Testing Strategy
- **Unit Tests**: JUnit 5 with Spring Boot Test support
- **Integration Tests**: Use `@SpringBootTest` annotation
- **Test Coverage**: Aim for ≥80% unit test coverage
- **Test Location**: Mirror main source structure in `src/test/java`

### Spring Boot Configuration
- **Application Properties**: Located in `src/main/resources/application.properties`
- **Application Name**: OpsFinder
- **Profile Support**: Use Spring profiles for environment-specific configs

## Architecture Patterns

### Recommended Layers
```
┌─────────────────────┐
│   Controllers       │  ← REST/Web Layer
├─────────────────────┤
│   Services          │  ← Business Logic
├─────────────────────┤
│   Repositories      │  ← Data Access
├─────────────────────┤
│   Models/Entities   │  ← Domain Objects
└─────────────────────┘
```

### Best Practices
- **Dependency Injection**: Use constructor injection for better testability
- **Service Layer**: Keep business logic in @Service classes
- **Repository Pattern**: Use Spring Data repositories for data access
- **DTO Pattern**: Use DTOs for API requests/responses
- **Exception Handling**: Implement global exception handlers with @ControllerAdvice

## Claude Code Workflow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [siuyiuyeung/opsfinder](https://github.com/siuyiuyeung/opsfinder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
