---
trigger: always_on
description: Generates TypeScript `.d.ts` files from the JSON schemas in `dist/camel-catalog/`.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the Kaoto Camel Catalog Generator - a hybrid Java/TypeScript project that generates comprehensive Apache Camel catalogs for the Kaoto visual integration editor. The project uses Maven to build a Java-based catalog generator JAR, then uses Node.js/Yarn scripts to execute the generator and produce TypeScript type definitions from JSON schemas.

## Build Commands

### Full Build
```bash
yarn build
```
This runs the complete build pipeline:
1. Cleans dist and catalog directories
2. Builds the Maven JAR (`./mvnw clean package`)
3. Executes the catalog generator (Java JAR)
4. Generates TypeScript type definitions from JSON schemas
5. Copies generated catalog to the root `catalog/` folder

### Java-Only Build
```bash
yarn build:mvn
# or
./mvnw clean package
```
Builds the Java catalog generator JAR without running the full pipeline.

### Catalog Generation Only
```bash
yarn build:catalog
```
Runs the Java JAR to generate catalogs (requires JAR to be built first).

### TypeScript Type Generation
```bash
yarn build:ts
```
Generates TypeScript `.d.ts` files from the JSON schemas in `dist/camel-catalog/`.

### Linting
```bash
yarn lint          # Check for issues
yarn lint:fix      # Auto-fix issues
```

### Testing
```bash
./mvnw test                    # Run all tests
./mvnw test -Dtest=ClassName   # Run specific test class
```

## Architecture

### Two-Phase Build System

The build process is split into two distinct phases:

1. **Java Phase**: Maven builds a shaded JAR containing the catalog generator with all dependencies. This JAR is a command-line tool that:
   - Loads Apache Camel catalog metadata from Maven dependencies
   - Processes Kamelets (Camel connectors) from YAML definitions
   - Generates JSON schemas for Camel YAML DSL, CRDs, and XSD schemas
   - Produces a catalog index file referencing all generated artifacts
   - Main entry point: `io.kaoto.camelcatalog.Main`

2. **Node.js Phase**: Yarn scripts orchestrate post-processing:
   - Execute the Java JAR with configured Camel/Kamelets versions (via `scripts/build-catalogs.mjs`)
   - Generate TypeScript type definitions from JSON schemas (via `scripts/json-schema-to-typescript.mjs`)
   - Copy artifacts to the publishable `dist/` folder

### Catalog Generation Flow

The catalog generator (`CatalogGenerator.java`) follows this pipeline:
1. Load Camel catalog, Kamelets, Kubernetes schemas, and Camel-K CRDs
2. Process Camel YAML DSL schema (JSON Schema Draft-07)
3. Generate aggregated catalog JSON files for components, EIPs, and data formats
4. Process and aggregate Kamelet definitions
5. Generate schemas (YAML DSL, CRDs, XSD)
6. Create catalog index file with hash-based filenames for cache-busting

### Maven Artifact Loading and Resource Management

The project uses a dynamic Maven artifact resolution system to load versioned Camel catalogs at runtime without requiring them to be compile-time dependencies. This allows generating catalogs for multiple Camel versions from a single JAR.

#### KaotoMavenVersionManager

`KaotoMavenVersionManager` extends Apache Camel's `MavenVersionManager` and provides:

- **Dynamic Artifact Resolution**: Downloads Maven artifacts (JARs) from remote repositories at runtime
- **Transitive Dependency Support**: Unlike the base class, this resolves transitive dependencies needed for Quarkus and Spring Boot runtime providers
- **Custom ClassLoader**: Uses `KaotoOpenURLClassLoader` to dynamically add downloaded JARs to the classpath
- **Repository Configuration**: Automatically configures Maven Central and Red Hat GA repositories based on version string (detects `-redhat` suffix)
- **Version-Aware Resource Loading**: Ensures resources are loaded from the correct version when multiple Camel versions are on the classpath

Example artifact resolution (from CamelCatalogVersionLoader.java:252):
```java
// Downloads org.apache.camel.quarkus:camel-quarkus-catalog:3.27.0 and all dependencies
camelCatalog.loadRuntimeProviderVersion("org.apache.camel.quarkus",
                                        "camel-quarkus-catalog",
                                        "3.27.0");
```

This internally calls `KaotoMavenVersionManager.resolve()` which:
1. Uses `MavenDownloader` to fetch the artifact from configured repositories
2. Resolves transitive dependencies
3. Adds all JAR files to the custom URLClassLoader
4. Makes resources from those JARs available for loading

#### ResourceLoader

`ResourceLoader` provides utilities to extract resources from the dynamically loaded JARs:

- **Single Resource Loading** (`getResourceAsString`): Loads individual files like `camel-yaml-dsl.json` from the classpath
- **Bulk Folder Loading** (`loadResourcesFromFolderAsString`): Loads all files with a specific suffix from a folder in JARs or filesystem
  - Example: Load all `.kamelet.yaml` files from the `kamelets/` folder inside the kamelets JAR
  - Handles both JAR protocol (`jar:file:`) and file protocol (`file:`) resources
  - Returns a map of filename → content for processing

**How ResourceLoader handles JAR vs filesystem**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KaotoIO) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
