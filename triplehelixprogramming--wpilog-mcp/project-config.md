---
trigger: always_on
description: This document provides context and coding guidelines for Claude Code when working on wpilog-mcp.
---

# Claude Code Project Instructions

This document provides context and coding guidelines for Claude Code when working on wpilog-mcp.

## Project Overview

**wpilog-mcp** is a Java 17+ MCP server that parses WPILib robot telemetry logs (binary `.wpilog` format) and REV motor controller logs (`.revlog`), providing analysis tools for FRC (FIRST Robotics Competition) diagnostics via JSON-RPC 2.0 over stdio and HTTP transports.

Before working on this codebase, scan the package structure and read the key files relevant to your task. Do not rely on class names or counts mentioned in any documentation — verify against the actual code.

## Architecture at a Glance

The server has a layered architecture:

1. **Transport layer** — Accepts MCP JSON-RPC 2.0 messages over stdio (single-client) or HTTP Streamable (multi-client). Transport-independent message routing.
2. **Tool layer** — Dozens of analysis tools, each registered via a tool registry. Tools that need log data extend a common base class that auto-injects a `path` parameter and handles log loading. There is no "active log" concept — each tool call is self-contained.
3. **Log layer** — Lazy on-demand parsing of WPILOG files. A single-pass scan records entry metadata and byte offsets; values are decoded on demand via random access into the memory-mapped file and cached with Caffeine. An LRU cache manages loaded logs with idle expiration and heap-pressure-based eviction.
4. **RevLog layer** — REV motor controller log parsing (both WPILOG-format and REV native binary format) with DBC-based CAN signal decoding. Cross-correlation timestamp synchronization aligns revlog data to WPILOG FPGA timestamps.
5. **External integrations** — The Blue Alliance API for match data enrichment. Disk caching for expensive sync results. Named server configurations with environment variable interpolation.

## Java 17 Best Practices

All code should follow JDK 17 idioms:

- **Streams**: Prefer streams over imperative loops unless performance is a concern (e.g., hot paths, large datasets with measurable overhead).
- **`var`**: Use `var` when it aids readability by reducing redundancy (e.g., `var entries = map.entrySet()`). Never use `var` for primitive types (`int`, `long`, `double`, etc.).
- **Records**: Use records for immutable data carriers instead of classes with boilerplate getters/equals/hashCode.
- **Enums**: Prefer enums over string or integer constants for fixed sets of values.
- **Switch expressions**: Use switch expressions (`->` syntax) over traditional switch statements. Use pattern matching where applicable.
- **Sealed classes**: Use sealed classes/interfaces when a type hierarchy has a known, fixed set of subtypes.
- **`Optional`**: Use `Optional` for return types that may have no value. Never use `Optional` as a field or parameter type.
- **Text blocks**: Use text blocks (`"""`) for multi-line string literals.

## FRC Domain Knowledge

When working with FRC-specific code, keep these domain constants in mind:

- **Brownout thresholds**: 6.8V for roboRIO 1, 6.3V for roboRIO 2
- **Match phases**: Derived from DriverStation mode transitions (Enabled + Autonomous booleans). Handle FMS disabled gap between auto and teleop, practice mode without FMS, missing DS data.
- **Loop timing**: Auto-detect units (ms vs s) via median heuristic. Typical robot loop is 20ms (50Hz).
- **Swerve modules**: 4 modules (FL, FR, BL, BR) with drive velocity and steer angle per module.
- **CAN bus**: Distinguish disabled-state timeouts (normal) from enabled-state errors (problematic).

## Mathematical Rigor

This codebase performs numerical analysis on telemetry data. Maintain high standards for floating-point correctness:

- **Statistics**: Use Bessel's correction for sample standard deviation (n-1 denominator). Handle zero variance, single data points, all-NaN arrays.
- **Percentiles**: Correct interpolation at boundary conditions (0th, 100th percentile).
- **Derivatives**: Handle non-uniform timestamp spacing. Use appropriate smoothing windows.
- **Correlation**: Include sample sizes and p-values. Always caveat that correlation does not imply causation.
- **Edge cases**: Always handle empty datasets, single elements, NaN/Infinity propagation, division by zero, duplicate timestamps.

## LLM Epistemological Guardrails

This codebase implements strategies to prevent LLM overconfidence when interpreting telemetry data. When adding or modifying tools, follow these patterns:

### Primitive Tool Design
Return raw statistics (mean, std, percentiles, sample sizes, p-values) rather than pre-computed interpretations. Let the LLM synthesize across multiple tool calls rather than providing monolithic "health scores."

### Data Quality Metadata
Attach data quality scores to analysis results. Quality metrics include sample count, gap analysis, NaN ratio, and jitter.

### Analysis Directives
Generate analysis directives with:
- Confidence level (high/medium/low/insufficient) calibrated to data quality
- Guidance text using epistemic language ("suggests", "may indicate", "consistent with")
- Follow-up suggestions for additional analysis
- Single-match limitation warnings

### Tool Description Guidance

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TripleHelixProgramming/wpilog-mcp](https://github.com/TripleHelixProgramming/wpilog-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
