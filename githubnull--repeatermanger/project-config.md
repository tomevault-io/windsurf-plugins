---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Repeater Manager is a Burp Suite Professional extension that provides advanced HTTP request replay capabilities with persistent storage, history tracking, API extraction, privilege escalation testing, and enhanced organization features. The plugin is designed for security testers and penetration testers to efficiently manage and organize HTTP/HTTPS requests.

- **Version**: 2.16.2
- **Java**: 17 (source/target compatibility)
- **Build**: Maven
- **License**: Apache License 2.0

## Architecture

The project follows a layered architecture:
```
+---------------------+
|      UI Layer       |  Java Swing + RSyntaxTextArea
+---------------------+
|   Service Layer     |  AutoSave / GC / HistoryRecording / ApiExtraction / PrivilegeTest / ReportGeneration
+---------------------+
|   Data Access Layer |  RequestDAO / HistoryDAO / PoolManager / ApiExtractionRuleDAO
+---------------------+
|   Data Storage      |  SQLite + File Blobs (Pool Dedup) + YAML (Global Rules)
+---------------------+
```

Key components:
- **BurpExtender.java**: Entry point implementing Montoya SDK's `BurpExtension` interface, initialized via `initialize(MontoyaApi)`
- **MontoyaApiHolder.java**: Static holder for `MontoyaApi` instance, bridges legacy static access pattern to constructor injection
- **RepeaterManagerUI.java**: Main UI controller that orchestrates all components
- **DatabaseManager.java**: Singleton managing SQLite connections (connection pool via `BlockingQueue` + JDK dynamic proxy), Schema initialization, session management
- **PoolManager.java**: Content deduplication manager using SHA-256 hash-based pools
- **RequestManager.java**: Async HTTP request sender using `MontoyaApi.http().sendRequest()`
- **HistoryRecordingService.java**: Singleton async queue-based history recording service
- **GarbageCollectorService.java**: Background GC for zero-reference pool data cleanup
- **AutoSaveService.java**: Scheduled database checkpoint service
- **LogManager.java**: Multi-channel log dispatcher with level filtering (Burp console / rolling file / UI panel)
- **ErmArchiveWriter/Reader.java**: Custom binary archive format with optional AES-256-CBC + HMAC-SHA256 encryption
- **DatabaseConfig.java**: Configuration management (storage mode / logging / proxy)
- **PopMenu.java**: Context menu provider implementing Montoya SDK's `ContextMenuItemsProvider`
- **ApiExtractionEngine.java**: Stateless rule-based API extraction engine (regex/substring/JSONPath/XPath) from URL path, query, headers, body
- **GlobalRuleManager.java**: Singleton managing global API extraction rules stored in `~/.burp/repeater_manager/api_extraction_rules.yaml`
- **ApiRuleManager.java**: Project-level API extraction rule management with SQLite persistence
- **AutoTestEngine.java**: Automated privilege escalation testing from proxy-intercepted scope-matched traffic
- **ReplayEngine.java**: Handles request replay logic for privilege testing
- **JudgmentEngine.java**: Evaluates responses against configurable rules to detect privilege escalation
- **TokenReplacementEngine.java**: Manages token substitution in requests across user sessions
- **DiffEngine.java / DiffPane.java / SearchBar.java / DiffNavigator.java**: Message comparison components with string/byte-level diff, syntax highlighting, synchronized scrolling, and change navigation (ComparisonDialog for full-featured comparison)
- **ReportGenerator.java**: Abstract base class for report generation with PdfReportGenerator (Apache PDFBox), HtmlReportGenerator, and MarkdownReportGenerator (FreeMarker templates) implementations
- **ReportExporter.java**: Unified report export dispatcher (PDF/HTML/Markdown)
- **BodyRenderer.java / BinaryContentRenderer.java**: Request/response body rendering including binary content support
- **GlobalTokenLocationManager.java**: Singleton managing global token locations shared across sessions (TokenLocationYamlIO for YAML serialization)
- **UserSessionYamlIO.java**: YAML serialization for user session import/export
- **RequestDispatchHandler.java**: Central request dispatch handler coordinating privilege test and normal request flows
- **FileChooserHelper.java**: Unified file chooser utility for consistent file selection dialogs

## Key Features

1. **Request Management**: Organize and categorize HTTP requests with color marking and comments
2. **History Tracking**: Automatic recording of request response history for comparison
3. **Data Persistence**: All requests and history saved to SQLite database with Pool deduplication
4. **Content Deduplication**: Pool architecture (string_pool/header_pool/body_pool/file_pool) with ref_count tracking
5. **Advanced Search**: Multi-condition filtering to quickly locate requests/responses
6. **Column Display Control**: Customizable table columns for better information density
7. **Data Import/Export**: ERM encrypted archives and Postman Collection v2.1 support
8. **Auto-save**: Periodic database checkpoint synchronization
9. **Garbage Collection**: Background cleanup of zero-reference pool data (10min interval)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GitHubNull/repeaterManger](https://github.com/GitHubNull/repeaterManger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
