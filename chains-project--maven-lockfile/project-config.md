---
trigger: always_on
description: - What: A Maven plugin + supporting GitHub Action that generates, persists and validates a repository-level lockfile (JSON) containing resolved dependency graphs, checksums, plugin metadata and POM/BOM metadata. It supports generate, validate and freeze goals to produce lockfile.json, verify an on-disk lockfile, and produce a frozen pom (pom.lockfile.xml).
---

# Project Overview

Maven Lockfile

- What: A Maven plugin + supporting GitHub Action that generates, persists and validates a repository-level lockfile (JSON) containing resolved dependency graphs, checksums, plugin metadata and POM/BOM metadata. It supports generate, validate and freeze goals to produce lockfile.json, verify an on-disk lockfile, and produce a frozen pom (pom.lockfile.xml).
- Ecosystem: Java / Maven plugin. The main module is maven_plugin.
- Intended usage: add to CI to validate reproducibility and integrity of Maven builds; optionally commit regenerated lockfiles back to PRs; enable reproducible rebuilds by freezing POMs.

# Key Components

- action.yml: composite GitHub Action wrapper that runs the plugin in CI (generate vs validate), detects changed files, optionally commits updated lockfiles. **This file is generated from template/action.yml** by the gmavenplus-plugin during the `generate-resources` phase (configured in the parent pom.xml). Do not edit action.yml directly — edit template/action.yml instead and regenerate.
- template/action.yml: source template for action.yml; contains the runtime bash driver (helpers like execute_maven_command) and the `PLUGIN_VERSION` placeholder that is substituted during the generate-resources phase. Changes here are propagated to action.yml at build time.
- maven_plugin/pom.xml: module POM (packaging=maven-plugin) — authoritative build config for the plugin artifact.

Primary Java constructs (use these when changing behavior):
- maven_plugin/src/main/java/io/github/chains_project/maven_lockfile/GenerateLockFileMojo.java:execute
- maven_plugin/src/main/java/io/github/chains_project/maven_lockfile/ValidateChecksumMojo.java:execute
- maven_plugin/src/main/java/io/github/chains_project/maven_lockfile/FreezeDependencyMojo.java:execute
- maven_plugin/src/main/java/io/github/chains_project/maven_lockfile/AbstractLockfileMojo.java:getConfig, getChecksumCalculator, generateMetaInformation
- maven_plugin/src/main/java/io/github/chains_project/maven_lockfile/LockFileFacade.java:generateLockFileFromProject, getAllPlugins, constructRecursivePom
- maven_plugin/src/main/java/io/github/chains_project/maven_lockfile/graph/DependencyGraph.java:of, createDependencyNode, getDependencySet
- maven_plugin/src/main/java/io/github/chains_project/maven_lockfile/graph/DependencyNode.java (data model)
- maven_plugin/src/main/java/io/github/chains_project/maven_lockfile/data/LockFile.java:readLockFile
- maven_plugin/src/main/java/io/github/chains_project/maven_lockfile/checksum/AbstractChecksumCalculator.java (API)
- maven_plugin/src/main/java/io/github/chains_project/maven_lockfile/checksum/FileSystemChecksumCalculator.java
- maven_plugin/src/main/java/io/github/chains_project/maven_lockfile/checksum/RemoteChecksumCalculator.java
- maven_plugin/src/main/java/io/github/chains_project/maven_lockfile/JsonUtils.java:getGson, toJson, fromJson

# Architecture

High-level data flow:

    Developer/CI
        │
        ▼
    GitHub Action (action.yml / template)  ── invokes ──▶ mvn io.github.chains-project:maven-lockfile:generate|validate|freeze
                                                          │
                                                          ▼
                                              Maven Mojo (Generate/Validate/Freeze)
                                                          │
                           ┌──────────────────────────────┴──────────────────────────────┐
                           │                                                             │
                           ▼                                                             ▼
                  LockFileFacade.generateLockFileFromProject                         FreezeDependencyMojo
                           │                                                             │
                           ▼                                                             ▼
                  DependencyGraph.of(...)  ←── AbstractChecksumCalculator ──▶ checksum impls
                           │                                                             │
                           ▼                                                             ▼
                    LockFile (data model)  ── JsonUtils ▸ write/read lockfile.json   pom.lockfile.xml

# Core Data Structures

- maven_plugin/src/main/java/io/github/chains_project/maven_lockfile/data/LockFile.java
  - Root model serialized to lockfile.json. Contains: project Pom, Set<DependencyNode> dependencies, Set<MavenPlugin> mavenPlugins, MetaData (Environment + Config), Set<Pom> boms.
  - Use LockFile.readLockFile(Path) to deserialize.

- maven_plugin/src/main/java/io/github/chains_project/maven_lockfile/graph/DependencyNode.java

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chains-project/maven-lockfile](https://github.com/chains-project/maven-lockfile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
