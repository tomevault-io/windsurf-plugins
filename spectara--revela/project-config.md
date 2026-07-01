---
trigger: always_on
description: > **Detailed conventions live in scoped instruction files** under [`.github/instructions/`](./instructions/).
---

# GitHub Copilot — Revela

> **Detailed conventions live in scoped instruction files** under [`.github/instructions/`](./instructions/).
> They auto-apply via `applyTo:` globs. This file is the always-on baseline only — keep it short.

---

## Project Overview

**Revela** ([revela.website](https://revela.website)) is a **.NET 10 / C# 14** static site generator for photographers. It is a complete rewrite of the original Bash project [`Expose`](https://github.com/kirkone/Expose) — same output, modern internals.

| Aspect | Original (Bash) | Revela |
|--------|-----------------|--------|
| Templates | Mustache-Light (regex) | Scriban |
| Markdown | Perl | Markdig |
| Images | VIPS CLI | NetVips |
| EXIF | ExifTool | NetVips built-in |
| Plugins | ❌ | ✅ NuGet-based |

**Status:** Pre-release — **no users, no backward compatibility required.** Rename, restructure, refactor freely. No migration scripts, no deprecation warnings.

---

## Project Structure

```
src/
├── Sdk/              # Public abstractions for plugin/theme authors
├── Sdk.Generators/   # Roslyn source generators ([RevelaConfig], etc.)
├── Core/             # Shared kernel — services, package loading, configuration
├── Commands/         # Host-owned CLI commands (Config, Packages, Plugins, Restore)
├── Features/         # Always built-in (Generate, Theme, Projects) — NOT plugins
├── Plugins/          # External plugins (Calendar, Compress, Serve, Source/*, Statistics)
├── Themes/           # Lumina (base) + Lumina.Calendar, Lumina.Statistics (extensions)
├── Cli/              # Entry point — dynamic plugin loading (DiskPackageSource)
└── Cli.Embedded/     # Entry point — static plugin refs (EmbeddedPackageSource) ← F5 debug
tests/                # Mirrors src/ + Shared (fixtures)
samples/              # revela-website, showcase, onedrive, calendar
benchmarks/           # BenchmarkDotNet projects
docs/                 # Architecture, plugin dev, setup
scripts/              # build-release.ps1, test-release.ps1
```

**Two entry points, one bootstrap:** `Cli` and `Cli.Embedded` differ only in their `IPackageSource`. All shared setup lives in `src/Cli/Hosting/HostBootstrap.cs`.

---

## Custom Agents & Skills (use these!)

| Agent (`.github/agents/`) | When |
|---------------------------|------|
| **Revela Dev** | All implementation work — features, fixes, tests, refactoring |
| **Revela Reviewer** | Read-only audits — architecture, security, performance, conventions |
| **Spike Analyst** | New feature ideas — sharpens problem, weighs trade-offs, compares prior art, produces decision-ready spike report (read-only) |
| **Explore** | Fast read-only codebase exploration (subagent — call in parallel) |
| **Pattern Finder** | Subagent — finds 2-3 canonical examples to mirror; dispatched by Dev before implementing something new |
| **Convention Sentry** | Subagent — anti-pattern scanner (underscore fields, log interpolation, missing `StringComparison`, hardcoded paths) |
| **Plugin Auditor** | Subagent — single-plugin lifecycle/convention check; dispatch one per plugin in parallel |
| **Test Doctor** | Subagent — test-quality auditor (FluentAssertions/Moq leftovers, missing assertions, tautologies) |
| **Security Scout** | Subagent — OWASP scanner (secrets, vulnerable packages, path traversal, SSRF, weak crypto) |

| Skill (`.github/skills/`) | Purpose |
|---------------------------|---------|
| `build-sample` | Build sample projects with local CLI |
| `build-release` | Local release build (`scripts/build-release.ps1`) |
| `commit-changes` | Conventional Commits — wait for explicit user request |
| `create-release` | Bump version, update CHANGELOG, tag |
| `review-code` | Per-file code review against conventions |
| `test-release` | End-to-end release pipeline test |

| Instructions file | Scope |
|-------------------|-------|
| [`csharp.instructions.md`](./instructions/csharp.instructions.md) | All `*.cs` — naming, async, logging, modern C# |
| [`tests.instructions.md`](./instructions/tests.instructions.md) | `tests/**/*.cs` — MSTest v4, NSubstitute, fixtures |
| [`plugins.instructions.md`](./instructions/plugins.instructions.md) | `src/Plugins/**` — plugin lifecycle, CommandDescriptor |
| [`themes.instructions.md`](./instructions/themes.instructions.md) | `src/Themes/**` — Scriban templates, manifest, partials |

| Prompt file | Invocation |
|-------------|------------|
| `/full-review` | Multi-phase deep review → routes to Revela Reviewer |
| `/new-plugin` | Scaffold a new plugin → routes to Revela Dev |
| `/new-theme` | Scaffold a new theme → routes to Revela Dev |
| `/release-notes` | Generate CHANGELOG entries from commits |

---

## Hard Rules — Top 10 (most frequent agent mistakes)

These are enforced by `.editorconfig` (warnings → errors via `TreatWarningsAsErrors=true`). Full details in [`csharp.instructions.md`](./instructions/csharp.instructions.md).

1. **`var` everywhere** — never spell out the type.
2. **Private fields = `camelCase`** — NO underscore prefix.
3. **`StringComparison.Ordinal`** on every string method (except char overloads like `StartsWith('-')`).
4. **`CultureInfo.InvariantCulture`** on every formatting call.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Spectara/Revela](https://github.com/Spectara/Revela) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
