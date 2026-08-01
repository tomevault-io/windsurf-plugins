---
trigger: always_on
description: Goose is a powerful, fast, and scalable load testing framework built with Rust. This document provides guidelines for AI-assisted development and maintenance of the Goose project, including patterns for code implementation, documentation, and project management.
---

# Cline Rules for Goose Load Testing Framework

## Project Context
Goose is a powerful, fast, and scalable load testing framework built with Rust. This document provides guidelines for AI-assisted development and maintenance of the Goose project, including patterns for code implementation, documentation, and project management.

## Critical Implementation Notes

### Code Modification Guidelines
**Focus on requested changes only**:
- Only modify what was explicitly requested
- Do not rename variables for "consistency" unless asked
- Do not add error handling where it wasn't requested
- Do not refactor unrelated code during feature implementation
- Maintain existing code style and patterns

### Implementation Workflow
**Follow incremental development**:
1. Implement one feature or fix at a time
2. Test each change before moving to the next
3. Update relevant documentation with each change
4. Ensure backward compatibility unless breaking changes are explicitly approved

## Documentation Standards

### Comprehensive Documentation Locations
**Documentation exists in multiple places - ALL must be kept current**:
- **Primary docs**: `./src/docs/goose-book/src/` (mdBook documentation)
- **Project overview**: `README.md` (project introduction and quick start)
- **Change tracking**: `CHANGELOG.md` (user-facing changes and releases)
- **Development context**: `/memory-bank/` (AI development context and decisions)
- **Inline documentation**: All `*.rs` files (function docs, module docs, examples)
- **Code examples**: `/examples/` directory (working code samples)
- **Test documentation**: `tests/*.rs` (test descriptions and patterns)
- **Configuration help**: Built-in help text and error messages

### Automatic Documentation Updates
**Update documentation with code changes**:
- Search ALL documentation locations for references to changed functionality
- Verify accuracy by testing documented examples and configuration options
- Update inline documentation for any modified public APIs
- Ensure consistency across all documentation sources

### Memory Bank Maintenance
**Keep memory bank current**:
- Update `memory-bank/progress.md` when features are completed
- Refine `memory-bank/systemPatterns.md` with architectural changes
- Update `memory-bank/activeContext.md` with current development focus
- Document technical decisions in appropriate memory bank files

## Feature Development Guidelines

### New Feature Checklist
When implementing new features:
1. [ ] Create or reference GitHub issue
2. [ ] Document implementation plan
3. [ ] Follow existing code patterns
4. [ ] Add appropriate tests
5. [ ] Update documentation
6. [ ] Update CHANGELOG.md
7. [ ] Consider backward compatibility

### Testing Requirements
**Comprehensive test coverage**:
- Unit tests for new functionality
- Integration tests for feature interactions
- Document test patterns for complex features
- Ensure tests work with all feature flag combinations

## Project-Specific Patterns

### Report Generation
**Current reporting capabilities**:
- HTML reports with JavaScript charts (primary format)
- Markdown reports for text-based summaries
- JSON reports for programmatic processing
- PDF reports in development (Issue #628, using HTML-to-PDF via headless Chrome)

### Metrics Architecture
**Coordinated Omission Mitigation**:
- Recently implemented to correct for measurement artifacts
- Provides more accurate latency metrics
- Uses statistical corrections for dropped measurements
- Integration with all report formats

### Controller System
**Runtime control interfaces**:
- WebSocket controller for real-time control
- Telnet controller for command-line interaction
- Supports dynamic test adjustment
- Thread-safe communication patterns

### Gaggle (Distributed Testing)
**Currently being reimplemented**:
- Previous implementation disabled in v0.17.0
- Considering multiple approaches (Hydro, Zenoh, gRPC/Tonic)
- Will support distributed load generation
- Architecture decisions tracked in memory bank

## Configuration Management

### GooseConfiguration Pattern
**CLI and configuration file support**:
- Use `gumdrop` for CLI parsing
- Support both command-line and file-based configuration
- Validate options early with clear error messages
- Document all configuration options

### Default Values
**Use GooseDefault enum**:
- Centralized default value management
- Type-safe configuration defaults
- Easy to extend for new options

## Dependency Management

### Version Requirements
**Maintain compatibility**:
- Rust edition 2021
- Optional dependencies via feature flags
- Document minimum supported Rust version (MSRV)

### External Dependencies
**Be conservative with dependencies**:
- Prefer well-maintained crates with active communities
- Use feature flags for optional dependencies
- Document why each dependency is needed
- Consider security implications

## Context7 Configuration Maintenance

### Overview

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tag1consulting/goose](https://github.com/tag1consulting/goose) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
