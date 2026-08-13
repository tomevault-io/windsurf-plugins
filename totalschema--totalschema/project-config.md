---
trigger: always_on
description: TotalSchema is a **multi-connector deployment orchestration tool** for data engineering, managing deployments across databases, SSH servers, and infrastructure—not just a database migration tool.
---

# TotalSchema - AI Agent Guide

## Project Overview

TotalSchema is a **multi-connector deployment orchestration tool** for data engineering, managing deployments across databases, SSH servers, and infrastructure—not just a database migration tool. 
Key insight: **connectors** allow deployments to a wide variety of asset, not just databases.
Unlike other toolkits, TotalSchema does not force the notion of version: the user can opt for 
selecting an arbitrary subset of changes to deploy via wildcards.


## Architecture

### Custom IoC Container (Not Spring)
**TotalSchema uses a lightweight, reflection-free IoC container instead of Spring** for faster startup and predictable behavior.

- All components use `ComponentFactory<T>` discovered via Java `ServiceLoader` (not classpath scanning)
- Dependencies retrieved from `Context` in `newComponent()`, **never in constructor**
- Declare dependencies in `getRequiredContextTypes()` for validation
- No runtime reflection = faster CLI startup (~60ms vs Spring's ~2.5s)
- **📖 See:** `docs/developer/IOC_CONTAINER_ARCHITECTURE.md` for complete architecture details
- Example: `totalschema-core/src/main/java/io/github/totalschema/spi/ComponentFactory.java`

### Module Structure
```
totalschema-core/              → Engine, IoC, state management
totalschema-cli/               → Standalone distribution
totalschema-maven-plugin/      → Build integration
totalschema-extensions/        → Groovy scripting (provided scope)
totalschema-database-integrations/ → BigQuery, etc.
totalschema-release/           → Bundled distribution (CLI + extensions)
```

### Change Script Model
Files: `<order>.<desc>.<env>.<type>.<connector>.<ext>` (e.g., `0001.create_users.DEV.apply.mydb.sql`)
- `<connector>` must match `connectors` section in `totalschema.yml`
- Types: `apply` (once), `apply_always` (every run), `apply_on_change` (on content change), `revert` (rollback)
- Connectors: `jdbc`, `ssh-script`, `ssh-commands`, `shell`

## Critical Workflows

### Build & Test
```bash
mvn clean install                    # Full build with SpotBugs
mvn clean install -Dspotbugs.skip=true  # Fast iteration
mvn spotless:apply                   # Auto-fix formatting
```
**SpotBugs runs at compile phase**—add exclusions to `configuration/spotbugs-exclude.xml` with justification.

### Testing CLI Distribution
```bash
cd totalschema-release && mvn clean package
cd target && tar -xzf totalschema-*.tar.gz
cd totalschema-*/bin
./totalschema.sh apply -e DEV -w ../../../sample
```

### Integration Testing
Use `sample/` workspace with H2 databases. Config: `sample/totalschema.yml`

## Project-Specific Patterns

### 1. AbstractJdbcTableRepository Template Pattern
State/lock tables inherit from `AbstractJdbcTableRepository`:
```java
public MyRepository(SqlDialect dialect, Configuration config) {
    super(dialect, JdbcDatabaseFactory.getInstance(), "db-name", config);
}
protected String getDefaultTableName() { return "my_table"; }
protected String getDefaultCreateSql() { 
    return formatSqlWithTableName("CREATE TABLE %TABLE_NAME% (...)");
}
```
**Ref:** `docs/developer/REFACTORING_SUMMARY.md`, `totalschema-core/.../common/repository/AbstractJdbcTableRepository.java`

### 2. No Optional Arguments
Factories specify **exact** argument count/types. No defaults, no nulls:
```java
List<ArgumentSpecification<?>> getArgumentSpecifications() {
    return List.of(ArgumentSpecification.string("name"),
                   ArgumentSpecification.configuration("config"));
}
```
**Ref:** `docs/developer/SIMPLIFIED_ARGUMENT_VALIDATION.md`

### 3. Thread-Safe Closeables
Use `AtomicBoolean` with `compareAndSet()` for idempotent close:
```java
private final AtomicBoolean isClosed = new AtomicBoolean(false);
public void close() {
    if (!isClosed.compareAndSet(false, true)) return;
    // ... cleanup
}
```
**Ref:** `docs/developer/DEFAULTJDBCDATABASE_FINAL_REVIEW.md`

### 4. Lock Service (4-Layer Design)
- `DefaultDatabaseLockService` → orchestrator
- `LockRenewalPolicy` → TTL/renewal timing
- `DatabaseLockOperations` → DB I/O
- `ReentrantLockState` → in-memory state  
**Ref:** `docs/developer/LOCK_SERVICE_ARCHITECTURE.md` (comprehensive guide)

## Configuration & Secrets

### totalschema.yml Structure
- `stateRepository.type: database|csv` → tracks applied changes
- `lock.type: database` → optional distributed locking
- `variables` → global, overridden by `environments.<ENV>.variables`
- `connectors.<name>` → connector definitions (JDBC, SSH, shell)

### Variable Substitution
`${varName}` in any config value. Expressions evaluated via `ExpressionEvaluator`.

### Encrypted Secrets
```bash
totalschema.sh create-master-key       # generates totalschema-master.key
totalschema.sh encrypt-secret "pwd"    # outputs ${secret:!SECRET;1.0;BASE64...}
```
Use encrypted format in `totalschema.yml` variables.

## Testing

**Framework:** TestNG (NOT JUnit) + EasyMock/PowerMock
```java
@Test
public void testMyFeature() {
    Configuration mockConfig = createMock(Configuration.class);
    expect(mockConfig.getString("key")).andReturn(Optional.of("value"));
    replay(mockConfig);
    // ... test code ...

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [totalschema/totalschema](https://github.com/totalschema/totalschema) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
