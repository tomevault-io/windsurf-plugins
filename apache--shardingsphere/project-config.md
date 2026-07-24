---
trigger: always_on
description: *Professional Guide for AI Programming Assistants - Best Practices for ShardingSphere Code Development*
---

# CLAUDE.md - ShardingSphere AI Programming Guide

*Professional Guide for AI Programming Assistants - Best Practices for ShardingSphere Code Development*

## 🏗️ ShardingSphere Architecture Overview

### Project Overview
ShardingSphere is an ecosystem of distributed database solutions with JDBC driver, database proxy, and planned Sidecar modes.

### Core Module Architecture
```yaml
module_hierarchy:
  infrastructure_layer:
    - shardingsphere-infra: Common utilities, SPI definitions
    - shardingsphere-parser: SQL parsing (ANTLR4-based)
  engine_layer:
    - shardingsphere-mode: Configuration management
    - shardingsphere-kernel: Core execution engine
  access_layer:
    - shardingsphere-jdbc: Java JDBC driver
    - shardingsphere-proxy: Database proxy
  feature_layer:
    - shardingsphere-sharding: Data sharding
    - shardingsphere-encryption: Data encryption
    - shardingsphere-readwrite-splitting: Read/write splitting
```

### Technology Stack Decisions
- **ANTLR4**: SQL parsing and abstract syntax tree generation
- **Netty**: High-performance network communication (proxy mode)
- **Apache Calcite**: Query optimization and execution plans
- **SPI**: Plugin architecture for hot-pluggable extensions

### JDBC vs Proxy Patterns
- **JDBC**: Zero invasion, Java-only, highest performance
- **Proxy**: Language-agnostic, centralized management, advanced features

### Key Concepts
- **Sharding**: Horizontal data partitioning
- **DistSQL**: Distributed SQL for dynamic configuration
- **SPI Extension**: Algorithm, protocol, and execution extensions
- **Data Pipeline**: Migration and synchronization functionality

### Code Quality Standards
```yaml
self_documenting_code:
  method_naming: "10-15 characters, verb-noun patterns, no comments needed"
  examples: ["isValidEmailAddress()", "calculateOrderTotal()"]
  anti_examples: ["proc()", "getData()", "handle()"]

complex_logic:
  definition: "3+ nested levels or 20+ lines per method"
  handling: "Extract to meaningful private methods"

mock_boundaries:
  no_mock: "Simple objects, DTOs, stateless utilities"
  must_mock: "Database connections, network services, third-party interfaces"
  judgment: "Mock only with external dependencies or high construction cost"
```

## 🚀 AI Programming Best Practices

### How to Obtain High-Quality Code

#### 1. Source Code Development Request Template
```
Please implement [feature description] for [class name], requirements:
1. Follow ShardingSphere project coding standards and constraints
2. Use self-documenting programming, no comments
3. Extract complex logic into private methods
4. 100% test coverage unit tests
5. Pass spotless code formatting checks
6. Use @RequiredArgsConstructor constructor injection
```

#### 2. Unit Test Request Templates

**Basic Style-Consistent Testing**:
```
Please write unit tests for [class name], requirements:
1. Use ShardingSphere project testing style
2. Test method naming with assert*() prefix
3. Use Hamcrest assertion style assertThat(actual, is(expected))
4. Use Mockito for Mocking, follow project boundary principles
5. Maintain clear Given-When-Then structure
```

**Complex Tests for First-Pass Success**:
```
Please write complete unit tests for [complex class name], requirements:
1. First analyze the dependency relationships and complexity of the class under test
2. Identify all external dependencies that need Mocking
3. Gradually build test fixtures, ensure complete Mock chains
4. Write corresponding test methods for each branch
5. Use @BeforeEach to set up common Mocks
6. Use try-with-resources to manage MockedConstruction
7. Ensure all tests can run independently and pass

If you encounter uncertain dependency relationships, please ask me for confirmation.
```

**100% Coverage Testing**:
```
Please implement 100% test coverage for [specific class name] in shardingsphere-[module] module:
1. First generate coverage report to check current status:
   ./mvnw clean test jacoco:report -Djacoco.skip=false -pl [submodule]
   open [submodule]/target/site/jacoco/index.html
2. Identify all branches that need testing (red diamond markers)
3. Write multiple sets of test data for complex conditions
4. Ensure all exception paths have tests
5. Verify final coverage reaches 100%:
   ./mvnw test jacoco:check@jacoco-check -Pcoverage-check -Djacoco.check.class.pattern=[ClassName] -pl [submodule]

If dead code or uncovered branches are found, please explain in detail.
```

**Special Case Handling Testing**:
```
Please write unit tests for [class name], requirements:
1. Make every effort to achieve 100% coverage
2. If you encounter the following situations, please report to me:
   - Truly unreachable dead code (e.g., never-thrown exceptions)
   - Functions dependent on specific runtime environments (e.g., OS-specific functions)
   - Features requiring special hardware or network conditions
   - Protective programming code for extreme cases

Report format:
- Code location: [class name:line number]
- Uncoverage reason: [detailed explanation]
- Suggested solution: [if any]

Let me confirm before skipping coverage requirements for these codes.
```

**SQL Generation Class Testing**:
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/shardingsphere](https://github.com/apache/shardingsphere) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
