---
trigger: always_on
description: This file provides guidance for AI assistants and developers working with the openEQUELLA codebase.
---

# openEQUELLA Agent Instructions

This file provides guidance for AI assistants and developers working with the openEQUELLA codebase.
It is the single source of truth, shared by every AI coding agent — vendor-specific entry points such
as `CLAUDE.md` reference this file rather than duplicating it.

**For developers:** This document offers a comprehensive architectural overview and coding
conventions reference. For detailed development environment setup and contribution workflow, please
refer to [CONTRIBUTING.md](CONTRIBUTING.md).

## Project Overview

openEQUELLA is a digital repository platform that provides a single platform to house teaching and
learning, research, media, and library content. It is primarily developed and maintained
by [Edalex](https://edalex.com).

**Key Characteristics:**

- Runs on Java 21
- Runs within an embedded Tomcat server
- Supports Linux and Windows platforms
- Uses a hybrid architecture with both modern and legacy code
- Development primarily done in IntelliJ IDEA

## Technology Stack & Architecture

### Backend Technologies

**Languages:**

- **Scala** - Preferred for all new server-side code (Scala 2.13)
- **Java** - Used in legacy code and some specific contexts
- Mix of both throughout the codebase

**Frameworks & Libraries:**

- **Guice** - Primary dependency injection framework (use `@Inject` annotations)
- **Spring** - Used for supporting functions (less common than Guice)
- **JPF (Java Plugin Framework)** - Customized plugin system that structures the application into
  modules
- **Hibernate** - ORM for database persistence
- **JAX-RS** - REST endpoint definitions (use `@Path` annotations)
- **Apache Lucene** - File-based search index for content and ACL resolution

**Databases:**

- PostgreSQL (recommended for development)
- MS SQL Server
- Oracle

### Frontend Technologies

**New UI (Modern):**

- **TypeScript** - All new UI code
- **React** (v19.x) - UI framework
- **Material-UI (MUI)** (v7.x) - Component library (`@mui/material`)
- **fp-ts** - Functional programming utilities
- **io-ts** - Runtime type validation
- Located in `react-front-end/` directory
- REST communication handled by `oeq-ts-rest-api/` module

**Legacy UI:**

- **Sections Framework** - Bespoke Java-based web framework
- **Freemarker** - HTML templating (`.ftl` files)
- **Legacy JavaScript** - AJAX functionality in older pages
- Being gradually migrated to New UI

**Admin Console:**

- Java Swing application (standalone)
- Being migrated to New UI

### Build System

**Primary Build Tool: SBT**

- Main build configuration in `build.sbt`
- Extensive customization via SBT plugins in `project/` directory
- Custom AutoPlugins handle JPF-specific tasks:
  - `JPFScanPlugin` - Scans for plugin projects
  - `JPFPlugin` - Default settings for JPF plugins
  - `JPFRunnerPlugin` - Collects plugins for deployment/running
- Run `./sbt` from project root

**Frontend Build:**

- **NPM** - Package management (Node 24.11.0)
- **Vite**(v8.x) - Bundler for TypeScript/React code
- Build commands in `react-front-end/package.json`

**JPF Plugins:**

- Each plugin has a `plugin-jpf.xml` descriptor file
- Defines plugin dependencies and exports
- SBT automatically discovers and processes these

## Code Quality & Style Standards

### General Principles

When writing or modifying code:

- Follow **Clean Code** principles
- Prefer **declarative** over imperative idioms
- Use **pragmatic functional programming** patterns where appropriate
- Leverage **modern language features** (Scala 2.13, Java 21, ES2020+)
- Match conventions in similar existing code
- Document complex logic with clear comments
- Add ScalaDoc/JavaDoc/JSDoc for public APIs

### Language-Specific Guidance

**Scala:**

- Preferred for all new server-side code
- Use functional programming patterns (immutability, pure functions, etc.)
- Leverage Scala collections and for-comprehensions
- Follow conventions in existing Scala code

**Java:**

- Use for specific contexts or when maintaining existing Java code
- Follow modern Java practices (streams, Optional, etc.)
- Consider refactoring to Scala when making substantial changes

**TypeScript:**

- Required for all new UI code
- Use strict type checking
- Leverage fp-ts for functional patterns
- Use io-ts for runtime type validation
- Prefer functional components with hooks in React

### Code Formatting

**All code must be formatted using these tools:**

| Language              | Tool               | Files                   | Configuration                |
|-----------------------|--------------------|-------------------------|------------------------------|
| Scala                 | ScalaFmt           | `*.scala`, `*.sbt`      | `.scalafmt.conf` (v3.11.4)   |
| Java                  | Google Java Format | `*.java`                | Run via `google-java-format` |
| TypeScript/JavaScript | ESLint + Prettier  | `*.ts`, `*.tsx`, `*.js` | `eslint.config.mjs`          |
| CSS/SCSS              | Prettier           | `*.css`, `*.scss`       | Prettier defaults            |

**Pre-commit Hooks:**

- Install with `npm ci` in project root
- Automatically formats staged files before commit
- Uses `husky` and `lint-staged` (see `package.json`)

**Manual Formatting:**

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openequella/openEQUELLA](https://github.com/openequella/openEQUELLA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
