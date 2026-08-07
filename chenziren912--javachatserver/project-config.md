---
trigger: always_on
description: This document is the contributor guide for **ChatServer** — a self-contained,
---

# Repository Guidelines

This document is the contributor guide for **ChatServer** — a self-contained,
dependency-light Java social platform (chat, moments, groups, cloud drive,
music, video, mini-games, AI assistant, and admin tooling) built directly on
the JDK's built-in `com.sun.net.httpserver.HttpServer`. Read it before making
changes, and keep it up to date when architecture or conventions shift.

> For an exhaustive, machine-generated, file-by-file dump of the codebase, see
> [`项目详情.md`](./项目详情.md) (~3.4 MB, produced by `generate_agents.py`). That
> file is a raw reference; **this** file is the curated, human-facing guide.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Project Structure & Module Organization](#2-project-structure--module-organization)
3. [Runtime Data Layout (`chatserver/`)](#3-runtime-data-layout-chatserver)
4. [Build, Test, and Development Commands](#4-build-test-and-development-commands)
5. [Architecture Overview](#5-architecture-overview)
6. [Request Lifecycle & Routing](#6-request-lifecycle--routing)
7. [The Service Layer](#7-the-service-layer)
8. [The Model Layer](#8-the-model-layer)
9. [The Utility Layer](#9-the-utility-layer)
10. [Persistence & Storage Strategy](#10-persistence--storage-strategy)
11. [Frontend & Assets](#11-frontend--assets)
12. [HTTP API Surface](#12-http-api-surface)
13. [Security & Configuration](#13-security--configuration)
14. [Internationalization (i18n)](#14-internationalization-i18n)
15. [External Integrations](#15-external-integrations)
16. [Coding Style & Naming Conventions](#16-coding-style--naming-conventions)
17. [Testing Guidelines](#17-testing-guidelines)
18. [Commit & Pull Request Guidelines](#18-commit--pull-request-guidelines)
19. [Feature Domains Reference](#19-feature-domains-reference)
20. [Adding a New Feature: Step-by-Step](#20-adding-a-new-feature-step-by-step)
21. [Operational Notes & Gotchas](#21-operational-notes--gotchas)
22. [Agent-Specific Instructions](#22-agent-specific-instructions)

---

## 1. Project Overview

ChatServer is a **monolithic Java 17 application** that serves both the HTTP API
and the server-rendered HTML frontend from a single process. It deliberately
avoids heavyweight frameworks (no Spring, no servlet container, no ORM). The
entire web stack is the JDK's `HttpServer`, and all state is persisted as plain
JSON files plus content-addressed binary blobs under a local `chatserver/`
directory.

Key characteristics:

- **Zero external database.** State lives in JSON files and hashed blob files.
- **Singleton services.** Each domain is a `getInstance()` singleton holding an
  in-memory index (usually a `ConcurrentHashMap`) backed by periodic atomic
  writes to disk.
- **Thin entry handler + domain delegates.** `RequestHandler` implements the
  single `HttpHandler` registered on context `/`, owns authentication and route
  dispatch, then delegates pages, AI, cloud, mini-program, and admin requests to
  focused package-private handlers.
- **Server-rendered pages + rich client JS.** HTML shells are built as Java
  strings; the interactive SPA behavior lives in `src/main/resources/assets/`.
- **Chinese-first product.** User-facing strings and code comments are primarily
  Simplified Chinese; an `en` locale is provided via `switch`-based translation.

The build produces a single fat JAR you can launch with `java -jar`.

---

## 2. Project Structure & Module Organization

```text
ChatServer/
├── pom.xml                      # Maven build (Java 17, fat-jar assembly)
├── AGENTS.md                    # This contributor guide
├── 项目详情.md                  # Auto-generated exhaustive code dump (do not hand-edit)
├── generate_agents.py           # Script that regenerates 项目详情.md
├── src/
│   ├── main/
│   │   ├── java/com/chat/
│   │   │   ├── Main.java                 # Entry point (port resolution)
│   │   │   ├── server/                   # HTTP layer
│   │   │   │   ├── ChatHttpServer.java   # Server bootstrap + thread pool
│   │   │   │   ├── RequestHandler.java   # HTTP lifecycle + route dispatch (<4k lines)
│   │   │   │   ├── RequestHandlerSupport.java # Shared parsing/response/upload helpers
│   │   │   │   ├── AppPageRenderer.java  # Login/share/status/main HTML shells
│   │   │   │   ├── AiRequestHandler.java # AI endpoints + provider integration
│   │   │   │   ├── CloudRequestHandler.java # Cloud-drive endpoints
│   │   │   │   ├── GameRequestHandler.java  # Mini-program endpoints
│   │   │   │   ├── AdminRequestHandler.java # Admin and moderation endpoints
│   │   │   │   ├── MusicRequestHandler.java # Music, comments, metadata, and ZIP import
│   │   │   │   ├── CloudEntryMapper.java # Cloud response DTO mapping
│   │   │   │   ├── UserRoles.java        # Shared server-side role predicates
│   │   │   │   ├── StoredFileAccess.java # Unified stored-file authorization policy
│   │   │   │   ├── I18n.java             # Server-side translation helper
│   │   │   │   └── TestGen.java          # Dev/test scaffolding helper
│   │   │   ├── service/                  # Business logic (singletons)
│   │   │   │   ├── UserService.java
│   │   │   │   ├── MessageService.java
│   │   │   │   ├── CloudService.java     # Largest service (~1.5k lines)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chenziren912/JavaChatServer](https://github.com/chenziren912/JavaChatServer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-02 -->
