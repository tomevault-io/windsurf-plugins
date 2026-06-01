---
trigger: always_on
description: **Operaton** is a native BPMN 2.0 process engine that runs inside the Java Virtual Machine. It's a fork of Camunda 7 BPM platform and provides a complete stack for process automation including web applications for human workflow management, operations, and monitoring.
---

# Operaton Repository - Copilot Instructions

## Repository Overview

**Operaton** is a native BPMN 2.0 process engine that runs inside the Java Virtual Machine. It's a fork of Camunda 7 BPM platform and provides a complete stack for process automation including web applications for human workflow management, operations, and monitoring.

- **Repository Size**: Large multi-module Maven project (~35+ modules)
- **Languages**: Java 17+, JavaScript/TypeScript (Angular 1.8.x), HTML/CSS, SQL
- **Primary Frameworks**: Spring Boot 3.5.5, Quarkus 3.26.0, Angular 1.8.x, MyBatis 3.5.19
- **Database Support**: H2, PostgreSQL, MySQL, MariaDB, Oracle, SQL Server, DB2
- **Runtime Containers**: Tomcat, WildFly, Standalone (Operaton Run)

## Build Instructions

### Prerequisites

- **JDK 17 or newer** (required) - verify with `java -version`
- **Maven 3.9.0+** (Maven Wrapper provided - use `./mvnw` or `mvnw.cmd`)
- **Node.js v20+** and **npm 10+** (for frontend builds)

### Essential Build Commands

**Always run commands from the repository root** - the Maven Wrapper requires this for multi-module projects.

#### Core Build Commands
```bash
# Full build (includes tests and frontend) - ~15-20 minutes
./mvnw clean install

# Fast build (skip tests and frontend) - ~5-8 minutes  
./mvnw clean install -DskipTests -Dskip.frontend.build=true

# Build specific modules (after dependencies built)
./mvnw clean install -pl engine -DskipTests

# Clean all modules
./mvnw clean

# Clean up code to common coding standards using OpenRewrite and rules in `rewrite.yml`- ~12 minutes
.devenv/scripts/maintenance/code-cleanup.sh
```

#### Critical Build Order Dependencies
The engine module **requires** these dependencies built first:
```bash
# Build core dependencies first if building selectively
./mvnw clean install -pl parent,bom,commons,model-api,engine-dmn,juel -DskipTests
```

### Testing Commands

```bash
# Run all tests (can take 30+ minutes)
./mvnw test

# Run specific test categories  
./mvnw test -Dtest.includes=bpmn -pl engine
./mvnw test -Dtest.excludes=bpmn.async -pl engine

# Use predefined test profiles
./mvnw test -PtestBpmn -pl engine
./mvnw test -PtestExceptBpmn -pl engine
```

### Maven Profiles for Integration Testing

Compose profiles for **runtime container + testsuite + database**:

```bash
# Engine integration tests with H2
..devenv/scripts/build/build-and-run-integration-tests.sh

# Webapp integration tests with PostgreSQL
.devenv/scripts/build/build-and-run-integration-tests.sh --distro=wildfly --testsuite=webapps --db=postgresql

# Database testing with Testcontainers
./mvnw test -Ppostgresql,testcontainers -pl engine
```

**Available Profiles:**
- **Runtime**: `tomcat`, `wildfly`
- **Testsuite**: `engine-integration`, `webapps-integration`  
- **Database**: `h2`, `postgresql`, `mysql`, `oracle`, `sqlserver` (only H2 and PostgreSQL for engine-integration)

### Frontend Build Commands

```bash
cd webapps/frontend

# Install dependencies
npm install

# Build for production
npm run build

# Development server
npm start

# Linting and formatting
npm run lint
npm run prettier
```

### Common Build Issues & Solutions

#### 1. Network Download Failures
**Problem**: `chromedriver.storage.googleapis.com: No address associated with hostname`
**Solution**: Network connectivity issue with external dependencies. Skip QA modules:
```bash
./mvnw clean install -DskipTests -pl '!qa'
```

#### 2. Compilation Errors in Engine Module
**Problem**: `package org.operaton.bpm.dmn.engine does not exist`
**Solution**: Missing dependencies. Build dependencies first:
```bash
./mvnw clean install -pl parent,bom,commons,model-api,engine-dmn,juel -DskipTests
./mvnw clean install -pl engine -DskipTests
```

#### 3. Frontend Linting Issues
**Problem**: ESLint JSON formatting errors
**Solution**: Run prettier to fix formatting:
```bash
cd webapps/frontend && npm run prettier
```

#### 4. Out of Memory Errors
**Solution**: Increase Maven memory:
```bash
export MAVEN_OPTS="-Xmx4g -XX:MaxMetaspaceSize=1g"
```

#### 5. Resuming Failed Builds
**Problem**: Build fails partway through large multi-module build
**Solution**: Use Maven reactor resume:
```bash
# Maven will suggest the resume command after failure, e.g.:
./mvnw <args> -rf :operaton-engine
```

## Camunda 7 backports

For issues labeled with 'backport:c7' the following rules apply:

The backporting commit needs proper attribution of the original commit, including references to the backported commit, its original author and reference to the original issue.
The commit body should be formed like:
```

Related to https://github.com/camunda/camunda-bpm-platform/issues/{ISSUE}

Backported commit {HASH} from the camunda-bpm-platform repository.
Original author: {AUTHOR_NAME} <{AUTHOR_EMAIL}>
```

Squash changes to a single commit.

### Namespace change
Usually the changed files are located in the same module path, except for having operaton instead of camunda in their package name.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [operaton/operaton](https://github.com/operaton/operaton) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
