---
trigger: always_on
description: This is `Aregowe_PolyShellProtection`, a **high-risk ecommerce security module** for Adobe Commerce / Magento Open Source. It closes the PolyShell unrestricted file upload vulnerability (APSB25-94) via defense-in-depth: eight layered Magento plugins and three security models that block polyglot file attacks and webshell uploads across multiple interception points.
---

# Copilot Instructions — Aregowe_PolyShellProtection

## Project Identity

This is `Aregowe_PolyShellProtection`, a **high-risk ecommerce security module** for Adobe Commerce / Magento Open Source. It closes the PolyShell unrestricted file upload vulnerability (APSB25-94) via defense-in-depth: eight layered Magento plugins and three security models that block polyglot file attacks and webshell uploads across multiple interception points.

**Treat every change as a production security patch.** Correctness is more important than speed. A missed edge case can result in remote code execution on a merchant's server.

---

## Mandatory Review-First Collaboration Protocol

**Do not write code without explicit approval.** For every issue or recommendation:

1. Explain the concrete tradeoffs.
2. Give an opinionated recommendation.
3. Ask for input before assuming a direction.

### Per-Issue Response Format

For every specific issue (bug, smell, design concern, or risk):

- Describe the problem concretely, with file and line references.
- Present 2–3 options, including "Do Nothing" where reasonable.
- For each option: implementation effort, risk, impact on other code, maintenance burden.
- Give a recommended option and why, mapped to the engineering preferences below.
- Explicitly ask whether the user agrees or wants a different direction before proceeding.

### Review Workflow

When asked to review, work through sections interactively (Architecture → Code Quality → Tests → Performance), pausing after each section. Number issues, use lettered options per issue, and label recommendations clearly.

---

## Engineering Preferences

These guide all recommendations and code generation:

- **DRY is important.** Flag repetition aggressively. Shared constants already live in `FileUploadGuard` (e.g., `ALLOWED_IMAGE_EXTENSIONS`, `MIME_EXTENSION_MAP`, `BLOCKED_EXTENSION_PATTERN`, `BASE_ALLOWED_EXTENSIONS`, `BASE_BLOCKED_EXTENSIONS`). Shared utility methods like `FileUploadGuard::inferExtensionFromMimeType()`, `FileUploadGuard::inferExtensionForFileName()`, `FileUploadGuard::getAllowedExtensions()`, and `FileUploadGuard::getBlockedExtensions()` centralize cross-plugin logic. Never duplicate them.
- **Follow modern code standards** for PHP 8.1-compatible code, XML, JSON, and Adobe Commerce conventions.
- **Well-tested code is non-negotiable.** Favor more tests over fewer. Every public method, every edge case, every failure path.
- **Keep code engineered enough.** Avoid under-engineered fragility and over-engineered premature abstraction.
- **Err on the side of handling more edge cases, not fewer.** Thoughtfulness over speed.
- **Bias toward explicit over clever.** No magic, no implicit behavior, no "cute" one-liners that sacrifice readability.
- **Prefer minimal, targeted changes** with strong backward compatibility.

---

## Architecture Overview

### Defensive Layers

The module implements four defensive layers, each blocking attacks at a different depth:

| Layer | Purpose | Plugins | Sort Order |
|-------|---------|---------|------------|
| 1 — Request Path Blocking | Block HTTP requests to vulnerable media paths before routing | `BlockSuspiciousMediaPathPlugin`, `BlockSuspiciousMediaAppPathPlugin` | 5 |
| 2 — Controller & Upload Blocking | Block file upload controllers and file processor methods for dangerous entity types | `BlockCustomerAttributeFileUploadControllerPlugin`, `BlockCustomerFileUploadPlugin` | 1 |
| 3 — Custom Option Upload Validation | Filename validation and deep content scanning on custom option file uploads via REST API | `ValidateUploadedFileNamePlugin`, `ValidateUploadedFileContentPlugin`, `ValidateCustomOptionUploadPlugin` | 5, 10 |
| 4 — Framework-Level Image Hardening | Extension allowlisting, double-extension detection, polyglot scanning at the framework image validator/processor | `HardenImageContentValidatorPlugin`, `HardenImageProcessorPlugin` | 10 |

### Model Dependency Graph

```
FileUploadGuard (orchestrator)
├── PolyglotFileDetector (image + embedded PHP detection)
├── AttackPatternDetector (known filenames / regex patterns)
├── ScopeConfigInterface (admin-configured extension allowlist/blocklist)
│
└── Used by: ValidateUploadedFileNamePlugin,
             ValidateUploadedFileContentPlugin,
             ValidateCustomOptionUploadPlugin,
             HardenImageContentValidatorPlugin,
             HardenImageProcessorPlugin

SecurityPathGuard (path prefix blocking)
└── Used by: BlockSuspiciousMediaPathPlugin,
             BlockSuspiciousMediaAppPathPlugin

SecurityLogSanitizer (log context sanitization)
└── Used by: all plugins that log
```

### Fail-Closed vs Fail-Open Decisions

- **Path blocking plugins:** Fail-closed. If reflection or detection fails, the request is blocked.
- **Media app path plugin:** Fail-open with warning log. If reflection on `Media::$relativeFileName` fails, the request passes but a warning is logged. Other layers enforce restrictions.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aregowe/magento2-module-polyshell-protection](https://github.com/aregowe/magento2-module-polyshell-protection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
