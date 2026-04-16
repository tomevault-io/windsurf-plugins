---
trigger: always_on
description: This skill is not exhaustive. Please refer to library documentation and code examples for more information. Context7 can help as a discoverability platform.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Claude Code plugin containing AI agent skills covering engineering, marketing, and productivity domains. The repository provides reusable skill definitions that Claude Code can invoke across projects.

## Project Structure

```
skills/               # Claude Code skill definitions
  <skill-name>/
    SKILL.md          # Required: metadata + instructions
    references/       # Optional: detailed documentation loaded on demand
    scripts/          # Optional: executable code
    assets/           # Optional: templates, resources, config files, etc.
.claude-plugin/       # Plugin metadata and configuration
.cursor-plugin/       # Plugin metadata and configuration (version must match .claude-plugin/plugin.json)
gemini-extension.json # Gemini CLI extension manifest (version must match .claude-plugin/plugin.json)
```

## Agent Skills Specification

All skills MUST conform to the [Agent Skills specification](https://agentskills.io/specification.md). Key requirements are summarized below; the spec is the source of truth when in doubt.

## Frontmatter

New skills go in `skills/<skill-name>/SKILL.md`. Each SKILL.md has YAML frontmatter. Fields per the [Agent Skills spec](https://agentskills.io/specification.md) — **this project requires all fields marked "Project-required"**:

| Field | Required | Constraints |
| --- | --- | --- |
| `name` | Spec-required | 1-64 chars. Lowercase `a-z`, digits, hyphens. No leading/trailing/consecutive hyphens. **Must match parent directory name.** |
| `description` | Spec-required | 1-1024 chars. Describes what the skill does **and when to use it** — this is the primary triggering mechanism. Be specific and slightly "pushy" to avoid under-triggering. |
| `license` | Project-required | License name or reference to a bundled license file. Use `MIT` for this project. |
| `compatibility` | Project-required | 1-500 chars. Describe actual requirements. Base: `Designed for Claude Code or similar AI coding agents.` Extend when needed: add `Requires git`, `Requires internet access`, `Requires Python 3.14+ and uv`, etc. Skills with no special requirements use the base string only. |
| `metadata` | Project-required | Must include `author` (string), `version` (semver `a.b.c` string, e.g. `"1.0.0"`), and `openclaw` (ClawHub discoverability fields — see below). |
| `user-invocable` | Project-required | Boolean. `true` for skills invocable as slash commands (e.g. `/skill-name`), `false` (default) for contextual skills that auto-trigger. |
| `allowed-tools` | Project-required | Space-delimited list of pre-approved tools. See "Allowed tools" below. |

**Choosing `user-invocable`:** Use `false` (contextual) for domain expertise that enriches any relevant conversation without being explicitly asked — code style, security patterns, brand voice, commit conventions. Use `true` (user-invocable) for multi-step workflows the user explicitly triggers — ghostwriting a post, running a full audit, generating a commit message.

Example frontmatter:

```yaml
---
name: skill-example
description: "Skill for X. Use when doing Y."
user-invocable: false
license: MIT
compatibility: Designed for Claude Code or similar AI coding agents. Requires git.
metadata:
  author: samber
  version: "1.0.0"
  openclaw:
    emoji: "🔧"
    homepage: https://github.com/samber/cc-skills
    install:
      - kind: brew
        formula: jq
        bins: [jq]
    requires:
      bins:
        - git
        - jq
    skill-library-version: "1.2.3"
allowed-tools: Read Edit Write Glob Grep Bash(git:*) Agent
---
```

### ClawHub metadata (`metadata.openclaw`)

All skills MUST include `metadata.openclaw` fields for [ClawHub](https://github.com/openclaw/clawhub) discoverability and dependency management. See the [ClawHub skill format spec](https://github.com/openclaw/clawhub/blob/main/docs/skill-format.md) for the full reference.

| Field | Required | Description |
| --- | --- | --- |
| `emoji` | Yes | Display emoji for the skill |
| `homepage` | Yes | URL to the skill's homepage or docs. Use `https://github.com/samber/cc-skills` for this project |
| `install` | When deps exist | Array of install specs for dependencies. Supported kinds: `brew`, `node`, `go`, `uv`. Each entry has `kind`, `package`/`formula`, and `bins` |
| `requires.bins` | When bins needed | CLI binaries that must be installed at runtime. Omit for pure content skills with no CLI dependencies |
| `skill-library-version` | Optional (when covering a library/framework) | Semver or release tag of the library/framework/platform the skill was written against (e.g. `"2.1.0"`). Required for skills that document a specific third-party project so staleness can be detected. Omit for generic/content skills with no versioned dependency. |

`install` describes _how_ to get a dependency. `requires.bins` declares _what_ must exist at runtime. A skill can have `requires.bins` without `install` (e.g. `git` — assumed pre-installed) or both (e.g. `promql` — installable via `go install`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/samber) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
