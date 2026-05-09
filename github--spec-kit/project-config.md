---
trigger: always_on
description: **GitHub Spec Kit** is a comprehensive toolkit for implementing Spec-Driven Development (SDD) - a methodology that emphasizes creating clear specifications before implementation. The toolkit includes templates, scripts, and workflows that guide development teams through a structured approach to building software.
---

# AGENTS.md

## About Spec Kit and Specify

**GitHub Spec Kit** is a comprehensive toolkit for implementing Spec-Driven Development (SDD) - a methodology that emphasizes creating clear specifications before implementation. The toolkit includes templates, scripts, and workflows that guide development teams through a structured approach to building software.

**Specify CLI** is the command-line interface that bootstraps projects with the Spec Kit framework. It sets up the necessary directory structures, templates, and AI agent integrations to support the Spec-Driven Development workflow.

The toolkit supports multiple AI coding assistants, allowing teams to use their preferred tools while maintaining consistent project structure and development practices.

---

## Integration Architecture

Each AI agent is a self-contained **integration subpackage** under `src/specify_cli/integrations/<key>/`. The subpackage exposes a single class that declares all metadata and inherits setup/teardown logic from a base class. Built-in integrations are then instantiated and added to the global `INTEGRATION_REGISTRY` by `src/specify_cli/integrations/__init__.py` via `_register_builtins()`.

```
src/specify_cli/integrations/
├── __init__.py            # INTEGRATION_REGISTRY + _register_builtins()
├── base.py                # IntegrationBase, MarkdownIntegration, TomlIntegration, YamlIntegration, SkillsIntegration
├── manifest.py            # IntegrationManifest (file tracking)
├── claude/                # Example: SkillsIntegration subclass
│   └── __init__.py        #   ClaudeIntegration class
├── gemini/                # Example: TomlIntegration subclass
│   └── __init__.py
├── windsurf/              # Example: MarkdownIntegration subclass
│   └── __init__.py
├── copilot/               # Example: IntegrationBase subclass (custom setup)
│   └── __init__.py
└── ...                    # One subpackage per supported agent
```

The registry is the **single source of truth for Python integration metadata**. Supported agents, their directories, formats, capabilities, and context files are derived from the integration classes for the Python integration layer.

---

## Adding a New Integration

### 1. Choose a base class

| Your agent needs… | Subclass |
|---|---|
| Standard markdown commands (`.md`) | `MarkdownIntegration` |
| TOML-format commands (`.toml`) | `TomlIntegration` |
| YAML recipe files (`.yaml`) | `YamlIntegration` |
| Skill directories (`speckit-<name>/SKILL.md`) | `SkillsIntegration` |
| Fully custom output (companion files, settings merge, etc.) | `IntegrationBase` directly |

Most agents only need `MarkdownIntegration` — a minimal subclass with zero method overrides.

### 2. Create the subpackage

Create `src/specify_cli/integrations/<package_dir>/__init__.py`, where `<package_dir>` is the Python-safe directory name derived from `<key>`: use the key as-is when it contains no hyphens (e.g., key `"gemini"` → `gemini/`), or replace hyphens with underscores when it does (e.g., key `"kiro-cli"` → `kiro_cli/`). The `IntegrationBase.key` class attribute always retains the original hyphenated value, since that is what the CLI and registry use. For CLI-based integrations (`requires_cli: True`), the `key` should match the actual CLI tool name (the executable users install and run) so CLI checks can resolve it correctly. For IDE-based integrations (`requires_cli: False`), use the canonical integration identifier instead.

**Minimal example — Markdown agent (Windsurf):**

```python
"""Windsurf IDE integration."""

from ..base import MarkdownIntegration


class WindsurfIntegration(MarkdownIntegration):
    key = "windsurf"
    config = {
        "name": "Windsurf",
        "folder": ".windsurf/",
        "commands_subdir": "workflows",
        "install_url": None,
        "requires_cli": False,
    }
    registrar_config = {
        "dir": ".windsurf/workflows",
        "format": "markdown",
        "args": "$ARGUMENTS",
        "extension": ".md",
    }
    context_file = ".windsurf/rules/specify-rules.md"
```

**TOML agent (Gemini):**

```python
"""Gemini CLI integration."""

from ..base import TomlIntegration


class GeminiIntegration(TomlIntegration):
    key = "gemini"
    config = {
        "name": "Gemini CLI",
        "folder": ".gemini/",
        "commands_subdir": "commands",
        "install_url": "https://github.com/google-gemini/gemini-cli",
        "requires_cli": True,
    }
    registrar_config = {
        "dir": ".gemini/commands",
        "format": "toml",
        "args": "{{args}}",
        "extension": ".toml",
    }
    context_file = "GEMINI.md"
```

**Skills agent (Codex):**

```python
"""Codex CLI integration — skills-based agent."""

from __future__ import annotations

from ..base import IntegrationOption, SkillsIntegration


class CodexIntegration(SkillsIntegration):
    key = "codex"
    config = {
        "name": "Codex CLI",
        "folder": ".agents/",
        "commands_subdir": "skills",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [github/spec-kit](https://github.com/github/spec-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
