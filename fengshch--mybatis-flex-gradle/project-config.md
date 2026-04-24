---
trigger: always_on
description: generateSchema: PUBLIC
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Gradle plugin for MyBatis Flex code generation. It wraps the MyBatis Flex codegen library to provide Gradle integration and includes Flyway database migration support.

Plugin ID: `io.github.fengshch.mybatis-flex-gradle-plugin`
Main class: `io.github.fengshch.mybatis.MyBatisFlexGradlePlugin`

## Build Commands

```bash
# Clean build
gradle clean

# Build the plugin
gradle :mybatis-flex-gradle-plugin:build

# Publish to local Maven repository
gradle :mybatis-flex-gradle-plugin:publishToMavenLocal

# Publish To Nexus Maven Repository
gradle :mybatis-flex-gradle-plugin:publishPluginMavenPublicationToNexus-releaseRepository    

# Run tests
gradle :mybatis-flex-gradle-plugin:test
```

## Project Structure

This is a composite build with the plugin in `mybatis-flex-gradle-plugin/` as an included build.

### Key Components

**Plugin Entry**: `MyBatisFlexGradlePlugin.java`
- Applies JavaBasePlugin
- Creates `mybatis` and `flyway` extensions
- **Loads configurations from mybatis-flex.yml** (if exists) before processing build.gradle configurations
- Registers generation and Flyway tasks
- Reads datasource config from YAML files (application.yml, mybatis-flex.yml)
- Supports Spring profiles: dev, test, default (via `-Pprofile=<name>` or PROFILE env var)

**Extension**: `MyBatisExtension.java`
- Provides `NamedDomainObjectContainer<GlobalConfigBuilder>` for multiple configurations
- Allows defining multiple generation profiles (e.g., "main", "secondary")

**Configuration Builders** (in `config/` package):
- `GlobalConfigBuilder` - main configuration container, holds all other builders
- `DataSourceConfigBuilder` - database connection settings
- `PackageConfigBuilder` - package structure and source directory
- `StrategyConfigBuilder` - table/column naming strategies
- `EntityConfigBuilder`, `MapperConfigBuilder`, `ServiceConfigBuilder`, `ServiceImplConfigBuilder`, `ControllerConfigBuilder`, `TableDefConfigBuilder`, `MapperXmlConfigBuilder` - code generation configs for each artifact type
- `FlywayConfigBuilder` - Flyway migration configuration
- `JavadocConfigBuilder` - javadoc generation settings
- `TemplateConfigBuilder` - custom template configuration

**Tasks**:
- `mybatisGenerate` (or `mybatis<Name>Generate` for named configs) - generates MyBatis Flex code
- Flyway tasks: `flywayClean`, `flywayBaseline`, `flywayMigrate`, `flywayUndo`, `flywayValidate`, `flywayInfo`, `flywayRepair` (prefixed with config name if not "main")

## Architecture

### Configuration Flow

1. Plugin creates `mybatis` extension with named configurations
2. **Loads configurations from `src/main/resources/mybatis-flex.yml`** if the file exists
3. In `afterEvaluate`, for each configuration:
   - Reads datasource config from YAML (supports spring.datasource structure)
   - Creates `mybatis<Name>Generate` task
   - Registers Flyway tasks
   - Adds custom source directories to Java source sets

### Code Generation Flow

1. `MyBatisGenerateTask` reads `GlobalConfigBuilder`
2. **Cleans the batis directory** (deletes the configured source directory to ensure fresh generation)
3. Creates HikariDataSource from `DataSourceConfigBuilder`
4. Builds `GlobalConfig` from all config builders
5. Detects database dialect from JDBC driver
6. Runs MyBatis Flex `Generator` to introspect schema and generate code
7. Generates: Entity, Mapper, Service, ServiceImpl, Controller, TableDef, MapperXml (based on enable flags)

### Datasource Configuration

The plugin reads datasource config from:
1. `src/main/resources/mybatis-flex.yml` (if exists)
2. `src/main/resources/application.yml` or profile-specific files (application-dev.yml, application-test.yml)

Expected YAML structure:
```yaml
spring:
  datasource:
    url: jdbc:...
    username: ...
    password: ...
    driverClassName: ... # or driver-class-name
```

For multiple configurations, use nested structure:
```yaml
spring:
  datasource:
    main:
      url: ...
    secondary:
      url: ...
```

### MyBatis Flex Configuration (mybatis-flex.yml)

You can define all MyBatis Flex configurations in `src/main/resources/mybatis-flex.yml` instead of in `build.gradle`. This provides a cleaner separation of configuration from build logic.

Example `mybatis-flex.yml`:
```yaml
spring:
  datasource:
    url: jdbc:h2:file:./data/h2db
    username: sa
    password: password
    driverClassName: org.h2.Driver

mybatis-flex:
  configurations:
    main:
      controllerGenerateEnable: true
      entityGenerateEnable: true
      mapperGenerateEnable: true
      serviceGenerateEnable: false
      serviceImplGenerateEnable: false
      tableDefGenerateEnable: true
      mapperXmlGenerateEnable: false

      packageConfig:
        sourceDir: src/batis/java
        basePackage: com.example.mybatis
        entityPackage: po
        mapperPackage: mapper
        servicePackage: service
        serviceImplPackage: service.impl
        controllerPackage: controller
        tableDefPackage: table
        mapperXmlPath: src/main/resources/mapper

      strategyConfig:
        tablePrefix: tb_
        logicDeleteColumn: deleted
        versionColumn: version
        generateForView: false
        generateSchema: PUBLIC

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fengshch) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
