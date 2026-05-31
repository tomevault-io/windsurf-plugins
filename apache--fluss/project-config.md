---
trigger: always_on
description: <!-- SPDX-License-Identifier: Apache-2.0
---

 <!-- SPDX-License-Identifier: Apache-2.0
      https://www.apache.org/licenses/LICENSE-2.0 -->

# Apache Fluss - AI Agent Coding Guide

AI coding guide for Apache Fluss with critical rules, patterns, and standards derived from codebase analysis and Checkstyle enforcement.

**Purpose:** This guide helps AI coding agents contribute to Apache Fluss by providing project-specific conventions, architectural patterns, and quality standards. It covers both code contribution (Sections 1-10) and deployment/setup guidance (Section 11).

**Sections:** 1. Critical Rules | 2. API Patterns | 3. Code Organization | 4. Error Handling | 5. Concurrency | 6. Testing | 7. Dependencies | 8. Configuration | 9. Serialization/RPC | 10. Module Boundaries | 11. Build & CI | 12. Git & Pull Requests | 13. AI Agent Boundaries

---

## 1. Critical Rules (MUST/NEVER)

**Enforced by Checkstyle** - violations will fail CI.

### Dependencies & Utilities

**FORBIDDEN imports** (use shaded versions - see Section 7):
```java
import com.google.common.*                              // → org.apache.fluss.shaded.guava.*
import com.fasterxml.jackson.*, org.codehaus.jackson.*  // → org.apache.fluss.shaded.jackson2.*
import io.netty.*                                       // → org.apache.fluss.shaded.netty4.*
import org.apache.arrow.*                               // → org.apache.fluss.shaded.arrow.*
import org.apache.zookeeper.*                           // → org.apache.fluss.shaded.zookeeper38.*
```

**MANDATORY utility substitutions:**
```java
// ❌ com.google.common.base.Preconditions  → ✅ org.apache.fluss.utils.Preconditions (import statically)
// ❌ com.google.common.annotations.VisibleForTesting  → ✅ org.apache.fluss.annotation.VisibleForTesting
// ❌ org.apache.commons.lang3.SerializationUtils  → ✅ org.apache.fluss.utils.InstantiationUtil
// ❌ Boolean.getBoolean("prop")  → ✅ Boolean.parseBoolean(System.getProperty("prop"))
```

### Java Version Compatibility

**Source level: Java 8** — All code MUST compile with JDK 8. CI enforces this via `compile-on-jdk8`.

**Build requirement:** Java 11 is required to build the project, but all source code must remain Java 8 compatible.

**FORBIDDEN Java 9+ features:**
```java
// ❌ var keyword (Java 10)
var list = new ArrayList<>();  // → ✅ ArrayList<String> list = new ArrayList<>();

// ❌ List.of(), Map.of(), Set.of() (Java 9)
List.of("a", "b")              // → ✅ Arrays.asList("a", "b")
Map.of("k", "v")               // → ✅ Collections.singletonMap("k", "v")
Set.of("a", "b")               // → ✅ new HashSet<>(Arrays.asList("a", "b"))

// ❌ Optional.isEmpty() (Java 11)
optional.isEmpty()             // → ✅ !optional.isPresent()

// ❌ String.strip(), String.isBlank() (Java 11)
string.strip()                 // → ✅ string.trim()
string.isBlank()               // → ✅ string.trim().isEmpty()

// ❌ Stream.toList() (Java 16)
stream.toList()                // → ✅ stream.collect(Collectors.toList())

// ❌ Map.entry() (Java 9)
Map.entry("k", "v")            // → ✅ new AbstractMap.SimpleEntry<>("k", "v")

// ❌ InputStream.transferTo() (Java 9)
inputStream.transferTo(out)    // → ✅ IOUtils.copyBytes(inputStream, out)

// ❌ Switch expressions, text blocks, records, sealed classes, pattern matching
```

**FORBIDDEN language features:** Switch expressions (Java 12), text blocks (Java 13), records (Java 14), sealed classes (Java 17), pattern matching (Java 16+)

### Testing

**MANDATORY: Use AssertJ, NOT JUnit assertions:**
```java
// ❌ Assertions.assertEquals(expected, actual)
// ✅ assertThat(actual).isEqualTo(expected)

import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assertions.assertThatThrownBy;

assertThat(list).hasSize(3).contains("a", "b");
assertThatThrownBy(() -> doSomething()).isInstanceOf(IllegalArgumentException.class);
```

**NEVER use @Timeout on tests** - rely on global timeout

### Code Style

- **NEVER use star imports** (`import java.util.*;`) - set IDE threshold to 9999
- **NEVER have trailing whitespace** - run `./mvnw spotless:apply`
- **ALWAYS use Java-style arrays:** `String[] args` NOT `String args[]`
- **ALWAYS require braces:** `if (x) { doIt(); }` NOT `if (x) doIt();`
- **NEVER use TODO(username)** - use `TODO:` without username
- **NEVER use FIXME, XXX, or @author tags** - use git history

### Documentation

- **File size limit:** 3000 lines max
- **Javadoc REQUIRED:** All protected/public classes, interfaces, enums, methods

---

## 2. API Design Patterns

### API Stability Annotations

```java
@PublicStable    // Stable - breaking changes only in major versions
@PublicEvolving  // May change in minor versions
@Internal        // Not public API - can change anytime
```

**Usage:** `@PublicStable` for core APIs (`Connection`, `Admin`); `@PublicEvolving` for new features; `@Internal` for RPC/internals
**Reference:** `fluss-common/src/main/java/org/apache/fluss/annotation/`

### Builder Pattern

```java
ConfigOption<Duration> timeout = ConfigBuilder
    .key("client.timeout")
    .durationType()
    .defaultValue(Duration.ofSeconds(30));
```

**Pattern:** Static inner `Builder` class, method chaining, private constructor, `build()` method
**Reference:** `fluss-common/src/main/java/org/apache/fluss/config/ConfigBuilder.java`

### Factory Pattern

```java

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/fluss](https://github.com/apache/fluss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
