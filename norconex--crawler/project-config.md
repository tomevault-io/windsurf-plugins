---
trigger: always_on
description: This is a Java-based Maven mono-repository containing the **Norconex Crawler v4 stack** - a comprehensive suite of web and file system crawlers with pluggable committers for various data stores. The crawler framework is designed to be highly configurable, extensible, and capable of handling complex crawling scenarios including document transformation, filtering, and routing to multiple backends.
---

# Norconex Crawlers - Copilot Instructions

## Repository Overview

This is a Java-based Maven mono-repository containing the **Norconex Crawler v4 stack** - a comprehensive suite of web and file system crawlers with pluggable committers for various data stores. The crawler framework is designed to be highly configurable, extensible, and capable of handling complex crawling scenarios including document transformation, filtering, and routing to multiple backends.

**Repository Type:** Multi-module Maven project (Java 21, Maven 3+)  
**Size:** ~15 modules, ~350 Java files  
**Target Runtimes:** Java 21+, Command-line applications, Embeddable libraries  
**Key Technologies:** Maven, Lombok, JaCoCo, JUnit 5, AssertJ, Jackson, Apache Commons

## High-Level Architecture

### Core Modules
- **crawler/core** - Shared crawler functionality and APIs
- **crawler/web** - Web crawling implementation 
- **crawler/fs** - File system crawling implementation
- **importer** - Document parsing and transformation engine

### Committer Modules (Data Output)
- **committer/core** - Base committer functionality
- **committer/elasticsearch** - Elasticsearch integration
- **committer/solr** - Apache Solr integration  
- **committer/sql** - SQL database integration
- **committer/neo4j** - Neo4j graph database integration
- **committer/amazoncloudsearch** - Amazon CloudSearch integration
- **committer/azurecognitivesearch** - Azure Cognitive Search integration
- **committer/apachekafka** - Apache Kafka integration

### Utility Modules  
- **tools/config-converter** - Configuration migration utilities

## Build and Validation Commands

### Prerequisites
**Always ensure Java 21+ and Maven 3+ are installed before building.**

### Bootstrap
```bash
# Clone and setup (no additional bootstrap required)
cd /path/to/crawlers
mvn --version  # Verify Maven 3+
java -version  # Verify Java 21+
```

### Build Commands
```bash
# Compile single module (RECOMMENDED for development)
mvn clean compile -pl committer/core

# Compile multiple related modules
mvn clean compile -pl committer/core,committer/solr

# Full reactor build (all modules)
mvn clean compile

# Full package with distribution (single module)
mvn clean package -pl committer/core -Dmaven.javadoc.skip=true

# Build with code coverage (activate JaCoCo profile)
mvn clean test -pl committer/core -Pjacoco
```

### Test Commands  
```bash
# Run tests for single module (FASTEST)
mvn test -pl committer/core

# Run tests with coverage reporting
mvn clean test -pl committer/core -Pjacoco

# Run specific test class
mvn test -pl committer/core -Dtest=AbstractBatchCommitterTest
```

### Validation Commands
```bash
# Format validation (Eclipse-based formatter)
mvn formatter:validate -pl committer/core

# Format fix  
mvn formatter:format -pl committer/core

# Editor config validation
mvn editorconfig:check -pl committer/core

# Dependency updates check
mvn versions:display-dependency-updates -pl committer/core
```

### Build Timings
- Single module compile: ~10-20 seconds
- Single module test: ~15-30 seconds  
- Single module package: ~20-60 seconds
- Full reactor build: ~90-120 seconds

### Known Build Issues & Workarounds

1. **Importer External Repository Requirement**: `importer` depends on `edu.ucar:jj2000:jar:5.4`, which is resolved from the OSGeo release repository declared in `importer/pom.xml`
   - **Current status**: The artifact resolves successfully from `https://repo.osgeo.org/repository/release/`
   - **Fallback**: If that external repository is temporarily unavailable, skip importer with `-pl '!importer'` for unrelated validation runs

2. **EditorConfig Warnings**: XML files may show indent warnings  
   - **Expected behavior**: Warnings don't fail the build

3. **Memory Requirements**: Use `-Xmx3g` for large builds
   - **Example**: `mvn clean package -pl crawler/web -DargLine="-Xmx3g"`

## Project Layout and Architecture

### Root Directory Structure
```
├── .editorconfig           # Code formatting rules (4 spaces for Java)
├── .github/workflows/      # CI/CD pipelines (Maven-based)
├── lombok.config          # Lombok configuration
├── norconex-formatter.xml # Eclipse formatter configuration  
├── pom.xml               # Parent POM with shared dependencies
├── sonar-project.properties # SonarCloud configuration
├── assembly/             # Maven assembly descriptors
├── crawler/              # Crawler implementations
├── committer/            # Data output implementations
├── importer/             # Document processing
└── tools/                # Utility tools
```

### Key Configuration Files
- **pom.xml** - Maven parent POM, Java 21, dependency versions
- **.editorconfig** - 4-space indentation, LF line endings
- **lombok.config** - Lombok settings (log field name, etc.)
- **norconex-formatter.xml** - Eclipse Java formatter settings

### Test Structure  
- Tests use **JUnit 5** with **AssertJ** assertions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Norconex/crawler](https://github.com/Norconex/crawler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
