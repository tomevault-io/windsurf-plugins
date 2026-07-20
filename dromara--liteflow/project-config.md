---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Overview

LiteFlow (v2.15.3) is a lightweight rules engine framework for complex component-based business orchestration. It uses a DSL to drive workflows with support for hot reload and 11 scripting languages. The project targets Java 8+ (up to JDK 25) and has 2000+ test cases.

**Official Documentation**: https://liteflow.cc/pages/5816c5/

## Commands

### Build
```bash
# Build entire project (uses 'compile' profile by default, includes test modules)
mvn clean package -DskipTests

# Build with tests
mvn clean package

# Build specific module
mvn clean package -DskipTests -pl liteflow-core

# Build for release (production modules only, excludes tests)
mvn clean package -DskipTests -P release
```

### Run Tests
```bash
# Run all tests
mvn test

# Run tests for specific module (30+ test modules in liteflow-testcase-el/)
mvn test -pl liteflow-testcase-el/liteflow-testcase-el-springboot

# Run single test class
mvn test -pl liteflow-core -Dtest=FlowExecutorTest

# Run specific test method
mvn test -pl liteflow-core -Dtest=FlowExecutorTest#testExecute
```

### Other Commands
```bash
# Dependency tree
mvn dependency:tree

# View module structure
ls liteflow-*/pom.xml
```

## High-Level Architecture

### Core Execution Model

**FlowExecutor** → **Chain** → **Condition Tree** → **Node Components**

1. **FlowExecutor**: Entry point for executing workflows (`execute2Resp(chainId, param)`)
2. **FlowBus**: Central metadata registry for all chains and nodes (thread-safe, supports hot reload)
3. **Chain**: Named workflow composed of an EL expression that compiles to a Condition tree
4. **Slot/DataBus**: Thread-safe context management using slot pooling (configurable `slotSize`)
5. **NodeComponent**: Base class for all business logic components

### Key Architectural Patterns

#### 1. Two-Stage Parsing
Chains are built in two phases to handle circular dependencies:
- **Phase 1**: Register chain IDs (creates placeholder chains)
- **Phase 2**: Build complete condition trees with EL parsing

#### 2. EL Expression Language
Uses QLExpress to parse declarative workflows. Example operators:
- **THEN(a, b, c)**: Sequential execution
- **WHEN(a, b, c)**: Parallel execution (async)
- **IF(condition, trueNode, falseNode)**: Conditional branching
- **SWITCH(selector).to(a, b, c)**: Multi-way branching
- **FOR(count).DO(loop)**: Fixed-count loop
- **WHILE(condition).DO(loop)**: Condition-based loop
- **ITERATOR(iterator).DO(loop)**: Iterator-based loop
- **RETRY(node).times(3).forException(Ex.class)**: Retry mechanism
- **CATCH(node).DO(handler)**: Exception handling
- **TIMEOUT(node).time(1000)**: Execution timeout (ms)
- **PRE(a, b)**: Pre-conditions (always run before chain, even on error)
- **FINALLY(a, b)**: Finally-conditions (always run after chain)
- **AND(a, b)**, **OR(a, b)**, **NOT(a)**: Boolean logic for IF conditions
- **node.tag("t")**, **.data("k","v")**, **.id("id")**: Per-node modifiers

Rules are defined in XML/JSON/YML:
```xml
<chain name="myChain">
    THEN(a, WHEN(b, c).maxWaitSeconds(5), IF(e, f, g));
</chain>
```

#### 3. Component Types
All extend `NodeComponent` but have specialized behaviors:
- **NodeComponent**: Standard synchronous component (`process()` method)
- **NodeBooleanComponent**: Returns boolean for IF conditions (`processBoolean()`)
- **NodeSwitchComponent**: Returns string for SWITCH routing (`processSwitch()`)
- **NodeIteratorComponent**: Provides iteration logic for ITERATOR construct
- **NodeForComponent**: Controls FOR loop behavior
- **ScriptComponent**: Script-based components (Groovy, JS, Python, etc.)

**Declarative Component Pattern**: Any Spring bean can become a component without extending base classes by using `@LiteflowCmpDefine` (class-level, specifies `NodeTypeEnum`) and `@LiteflowMethod` (method-level, maps to `LiteFlowMethodEnum`). This avoids class hierarchy constraints.

**Component Lifecycle Hooks** (override in NodeComponent subclasses or via `@LiteflowMethod`):
- `isAccess()` – gate check before execution; return `false` to skip
- `beforeProcess()` / `afterProcess()` – pre/post hooks per component
- `onSuccess()` / `onError()` – outcome callbacks
- `isContinueOnError()` – whether WHEN continues if this node fails
- `isEnd()` – signals chain should stop after this node
- `rollback()` – called in reverse order on failure

**`@FallbackCmp`**: Annotate a fallback component that activates when the primary component is not found or throws.

#### 4. Slot-Based Context
Thread-safe execution context:
- `DataBus.offerSlot(chainId)` acquires a slot from pool
- Slot contains execution metadata, context beans, and step tracking
- `DataBus.releaseSlot(slotIndex)` returns slot to pool
- Pool size configurable via `slotSize` property

#### 5. Parse Mode Strategies
Three modes (`ParseModeEnum`):
- **PARSE_ALL_ON_START**: Parse all chains at startup (default)
- **PARSE_ONE_ON_FIRST_EXEC**: Lazy parse each chain on first use (faster startup)
- **PARSE_ALL_ON_FIRST_EXEC**: Parse all chains on first any chain execution

### Module Structure

#### Core Modules

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dromara/liteflow](https://github.com/dromara/liteflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
