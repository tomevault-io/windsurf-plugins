---
trigger: always_on
description: **ALWAYS provide step-by-step reasoning that:**
---


# 🤖 **AI REASONING PROTOCOL**

## **Before Taking Any Architectural Action:**
**ALWAYS provide step-by-step reasoning that:**
1. States what architectural action you're about to take
2. Explains which rules from this document apply
3. Shows how you've considered those rules in your approach
4. Outlines your specific plan and next steps

---

# 🤖 **AI DEVELOPMENT RULES FOR REPOMAP-TOOL**

> **For AI (Cursor)**: These are the architectural rules you must follow when developing, modifying, or extending the RepoMap-Tool codebase.

## 🎯 **CORE ARCHITECTURAL PRINCIPLES**

### **🚨 FUNDAMENTAL: TREE-SITTER PARSING ARCHITECTURE**
- **THIS PROJECT USES TREE-SITTER DIRECTLY** for all code parsing and analysis
- **ALWAYS use tree-sitter parsers** for all code parsing and analysis
- **ALWAYS use TreeSitterParser.get_tags()** for code element extraction
- **PREFER get_tags() for caching benefits** - uses SQLite cache for performance
- **parse_file() is also valid** when caching is handled by DI container (standard pattern)
- **Performance note**: get_tags() provides 90%+ reduction in parse times for repeated operations
- **NEVER use Python's built-in ast.parse()** for production code analysis
- **NEVER use regex-based parsing** for code structure analysis
- **ALWAYS leverage tree-sitter's multi-language support** (Python, JS/TS, Java, Go, etc.)
- **ALWAYS use tree-sitter query files (.scm)** for parsing rules
- **NEVER reinvent tree-sitter functionality** - use the existing TreeSitterParser

### **Centralized Output Management**
- **ALL CLI output** must go through the `OutputManager` system
- **NO direct `console.print()` calls** in CLI commands (except utility functions)
- **Consistent formatting** across all commands and output types
- **Template-based rendering** for maintainable and flexible output

### **Dependency Injection (DI) Compliance**
- **ALL services** must use proper DI patterns
- **Constructor injection** is the preferred DI pattern for all services
- **NO direct instantiation** of services outside DI container
- **Service Factory pattern** for CLI commands (superior to @inject decorators)
- **Strict dependency validation** - no fallback instantiation allowed

### **Type Safety & Validation**
- **Full MyPy compliance** with comprehensive type annotations
- **Pydantic models** for all data structures and validation
- **Protocol-based interfaces** for extensibility and testability

### **🚨 CRITICAL: Absolute Path Standardization**
- **ALL file paths must be absolute** throughout the system (architectural requirement)
- **File discovery returns absolute paths** - no relative path conversion needed
- **Controllers work with absolute paths** - no path resolution redundancy
- **Only convert to relative paths** when calling tree-sitter parsing methods
- **PathResolver validates absolute paths** - raises errors for relative paths
- **Eliminates path confusion** and redundant conversions
- **Improves reliability** and performance

### **🚨 CRITICAL: Centralized Configuration Management**
- **ALL configuration values must use centralized ConfigService** - no hardcoded constants
- **ALWAYS use `get_config(key, default)` from `core.config_service`** for configuration access
- **NEVER hardcode thresholds, limits, timeouts, or other configurable values**
- **ALL configuration defaults defined in `ConfigDefaults` dataclass**
- **Configuration overrides supported via `set_config(key, value)`**
- **Global configuration service instance with singleton pattern**
- **Eliminates scattered hardcoded values** across 31+ files
- **Improves maintainability** - single place to change configuration

### **🚨 CRITICAL: Centralized Logging Management**
- **ALL logging must use centralized LoggingService** - no direct `logging.getLogger()`
- **ALWAYS use `get_logger(__name__)` from `core.logging_service`** for logger creation
- **NEVER use `logging.getLogger(__name__)` directly in modules**
- **Centralized logging configuration** with consistent formatting
- **Logger caching and singleton pattern** for performance
- **Global logging service instance** with auto-initialization
- **Eliminates duplicate logger setup** across 46+ files
- **Improves consistency** - unified logging behavior

### **🚨 CRITICAL: Configuration File Loading Requirements**
- **ALL CLI commands MUST properly load configuration files** - no exceptions
- **ALWAYS use `load_or_create_config()` from `cli.config.loader`** for configuration loading
- **NEVER use `create_analysis_config()` or `create_search_config()`** - these ignore config files
- **ALWAYS pass `config_file` parameter** to `load_or_create_config()`
- **NEVER bypass configuration file loading** with hardcoded factory methods
- **Configuration files MUST be respected** when `--config` parameter is provided
- **ALL commands must have consistent configuration loading behavior**
- **Eliminates configuration loading inconsistencies** across CLI commands
- **Improves user experience** - configuration files work as expected

### **AI Development Guidelines**
- **ALWAYS use tree-sitter parsers** - this is the core of the project, never bypass it
- **ALWAYS use TreeSitterParser.get_tags()** - never implement custom AST parsing
- **ALWAYS use tree-sitter query files (.scm)** for code element extraction

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xynova/repomap-tool](https://github.com/xynova/repomap-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
