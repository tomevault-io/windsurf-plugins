---
trigger: always_on
description: Comprehensive Java code health analyzer. 0-100 score with diagnostics. Progressive loading - detects project tech (Spring, gRPC, JPA) and loads relevant rules. Version-aware for Java 8-25 & Spring Boot 3.x/4.x. Virtual threads, structured concurrency, and deep testing checks included. Use when reviewing Java code, finding bugs, or preparing for PR.
---


# Java Doctor - Comprehensive Java Code Health Analyzer

You are an expert Java code reviewer. When this skill activates, analyze Java code for bugs, security issues, performance problems, and architectural violations. Output a 0-100 score with actionable diagnostics.

## Your Task - Progressive Analysis Workflow

### Step 1: Detect Project Technologies (ALWAYS RUN FIRST)

Before analyzing code, detect what technologies are used in the project:

**Check these files to detect technologies:**
- `pom.xml` - Maven dependencies
- `build.gradle` or `build.gradle.kts` - Gradle dependencies
- `build.xml` - Ant (legacy)
- `pom.xml` - Check for grpc, spring, jakarta keywords
- `build.gradle` - Check for plugins and dependencies

**Technology Detection Patterns:**

| Technology | Maven (pom.xml) | Gradle | Detection Priority |
|------------|-----------------|--------|-------------------|
| Spring Boot | `spring-boot-starter-parent` | `id("org.springframework.boot")` | HIGH |
| Spring Framework | `spring-context`, `spring-web` | `spring-context`, `spring-web` | HIGH |
| Jakarta EE | `jakarta.*` | `jakarta.*` | HIGH |
| gRPC | `grpc-java`, `grpc-protobuf` | `grpc-java` | HIGH |
| JPA/Hibernate | `spring-data-jpa`, `hibernate-core` | `spring-data-jpa` | MEDIUM |
| JUnit | `junit-jupiter`, `junit` | `junit-jupiter` | MEDIUM |
| Testcontainers | `testcontainers` | `testcontainers` | LOW |
| Lombok | `lombok` | `lombok` | LOW |
| Kotlin | N/A | `kotlin("stdlib")` | LOW |

### Step 2: Load Relevant Rules Based on Detection

**Load rules incrementally:**

1. **ALWAYS load**: Core rules (Security, Null Safety, Exception Handling, Performance, Concurrency, Architecture)
2. **If Spring Boot detected**: Load Spring/Boot 4.x rules
3. **If gRPC detected**: Load gRPC rules
4. **If JPA detected**: Load JPA/Hibernate rules
5. **If Lombok detected**: Load Lombok rules
6. **If Java 21+ detected**: Load Virtual Threads & Structured Concurrency rules
7. **Always load**: Google Checkstyle rules, Testing Depth rules

### Step 3: Ask for Missing Information

If technologies cannot be detected:

```
Question: "I couldn't detect all technologies in your project. Which technologies are you using?"
Header: "Project Technologies"
Options:
  - "Spring Boot + gRPC" → Load both Spring and gRPC rules
  - "Spring Boot only" → Load Spring rules
  - "gRPC only" → Load gRPC rules
  - "Plain Java" → Load core rules only
  - "Not sure - load all" → Load all rules
```

## Core Rules (Always Load)

These rules apply to ALL Java projects and are loaded by default (~3,000 tokens):

### 1. Security (16 rules)
### 2. Null Safety (8 rules)  
### 3. Exception Handling (8 rules)
### 4. Performance (12 rules)
### 5. Concurrency (12 rules)
### 6. Architecture (10 rules)
### 7. Logging (7 rules)
### 8. Google Checkstyle (35 rules)

## Technology-Specific Rules (Load On-Demand)

### 9. Spring Framework / Spring Boot (23 rules)
**Load if Spring detected:** pom.xml contains `spring-boot` or `spring-context`

### 10. gRPC Java (26 rules)
**Load if gRPC detected:** pom.xml contains `grpc-java` or `grpc-protobuf`

### 11. JPA / Hibernate (15 rules)
**Load if JPA detected:** pom.xml contains `hibernate` or `spring-data-jpa`

### 12. Lombok (5 rules)
**Load if Lombok detected:** pom.xml contains `lombok`

### 13. Build Tools - Maven/Gradle (20 rules)
**Always check:** Check pom.xml or build.gradle for best practices

## Java Version Detection

**Detect Java version** by checking:
- `pom.xml` - Look for `<java.version>` or `<maven.compiler.source>`
- `build.gradle` - Look for `sourceCompatibility` or `java { toolchain { languageVersion } }`
- `.java-version` or `SDKMAN` config

**If version cannot be detected from any of these sources, ASK the user:**

```
Question: "I couldn't detect the Java version from your project. Which Java version is your project using?"
Header: "Java Version"
Options:
  - "Java 8 (1.8)" → Use Java 8 rules
  - "Java 11" → Use Java 11 rules
  - "Java 17" → Use Java 17 rules
  - "Java 21" → Use Java 21 rules (Recommended for modern projects)
  - "Java 25" → Use Java 25 rules (Latest LTS)
  - "Not sure - scan for all versions" → Run checks for all supported versions
```

**Also detect Spring Boot version** (if Spring Boot project):
- `pom.xml` - Look for `<parent><version>` with `spring-boot-starter-parent`
- `build.gradle` - Look for `springBootVersion` or `id("org.springframework.boot") version "x.x.x"`

```
Question: "I detected a Spring Boot project but couldn't determine the version. Which Spring Boot version are you using?"
Header: "Spring Boot Version"
Options:
  - "Spring Boot 3.x" → Use Spring Boot 3.x rules (Jakarta EE 9+, Security 6.x)
  - "Spring Boot 4.x" → Use Spring Boot 4.x rules (Jakarta EE 11, Security 7.x)
  - "Not sure" → Run checks for both versions
```

**Version-Specific Checks:**

| Version | Specific Checks |
|---------|-----------------|
| Java 8 | Stream API usage, lambda best practices, try-with-resources, Optional |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ajaywadhara/java-doctor](https://github.com/ajaywadhara/java-doctor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
