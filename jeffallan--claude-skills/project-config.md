---
trigger: always_on
description: Use when encountering errors or unexpected behavior requiring investigation.
---

# Claude Skills Project Configuration

> This file governs Claude's behavior when working on the claude-skills repository.

---

## Skill Authorship Standards

Skills follow the [Agent Skills specification](https://agentskills.io/specification). This section covers project-specific conventions that go beyond the base spec.

### The Description Trap

**Critical:** Never put process steps or workflow sequences in descriptions. When descriptions contain step-by-step instructions, agents follow the brief description instead of reading the full skill content. This defeats the purpose of detailed skills.

Brief capability statements (what it does) and trigger conditions (when to use it) are both appropriate. Process steps (how it works) are not.

**BAD - Process steps in description:**
```yaml
description: Use for debugging. First investigate root cause, then analyze
patterns, test hypotheses, and implement fixes with tests.
```

**GOOD - Capability + trigger:**
```yaml
description: Diagnoses bugs through root cause analysis and pattern matching.
Use when encountering errors or unexpected behavior requiring investigation.
```

**Format:** `[Brief capability statement]. Use when [triggering conditions].`

Descriptions tell WHAT the skill does and WHEN to use it. The SKILL.md body tells HOW.

---

### Frontmatter Requirements

Per the [Agent Skills specification](https://agentskills.io/specification), only `name` and `description` are top-level required fields. Custom fields go under `metadata`.

```yaml
---
name: skill-name-with-hyphens
description: [Brief capability statement]. Use when [triggering conditions] - max 1024 chars
license: MIT
metadata:
  author: https://github.com/Jeffallan
  version: "1.0.0"
  domain: frontend
  triggers: keyword1, keyword2, keyword3
  role: specialist
  scope: implementation
  output-format: code
  related-skills: fullstack-guardian, test-master, devops-engineer
---
```

**Top-level fields (spec-defined):**
- `name`: Letters, numbers, and hyphens only (no parentheses or special characters)
- `description`: Maximum 1024 characters. Capability statement + trigger conditions. No process steps.
- `license`: Always `MIT` for this project
- `allowed-tools`: Space-delimited tool list (only on skills that restrict tools)

**Metadata fields (project-specific):**
- `author`: GitHub profile URL of the skill author
- `version`: Semantic version string (quoted, e.g., `"1.0.0"`)
- `domain`: Category from the domain list below
- `triggers`: Comma-separated searchable keywords
- `role`: `specialist` | `expert` | `architect` | `engineer`
- `scope`: `implementation` | `review` | `design` | `system-design` | `testing` | `analysis` | `infrastructure` | `optimization` | `architecture`
- `output-format`: `code` | `document` | `report` | `architecture` | `specification` | `schema` | `manifests` | `analysis` | `analysis-and-code` | `code+analysis`
- `related-skills`: Comma-separated skill directory names (e.g., `fullstack-guardian, test-master`). Must resolve to existing skill directories.

**Domain values:**
`language` · `backend` · `frontend` · `infrastructure` · `api-architecture` · `quality` · `devops` · `security` · `data-ml` · `platform` · `specialized` · `workflow`

---

### Reference File Standards

Reference files follow the [Agent Skills specification](https://agentskills.io/specification). No specific headers are required.

**Guidelines:**
- 100-600 lines per reference file
- Keep files focused on a single topic
- Complete, working code examples with TypeScript types
- Cross-reference related skills where relevant
- Include "when to use" and "when not to use" guidance
- Practical patterns over theoretical explanations

### Framework Idiom Principle

Reference files for framework-specific skills must reflect the idiomatic best practices of that framework, not generic patterns applied uniformly across all skills. If a framework provides a built-in mechanism (e.g., global error handling, middleware, dependency injection), reference examples should use it rather than duplicating that behavior manually. Each framework's conventions for error handling, architecture, and code organization take precedence over cross-project consistency.

---

### Documentation Backlink

Every `SKILL.md` MUST end with a single canonical Documentation link pointing back to the docs site:

```
[Documentation](https://jeffallan.github.io/claude-skills/skills/{domain}/{skill-name}/)
```

- `{domain}` is `metadata.domain` from the frontmatter (default `specialized` if absent — matches the fallback in `site/scripts/sync-content.mjs`).
- `{skill-name}` is the skill's directory name.
- Format is exactly markdown link syntax (`[text](url)`), not a bare URL. Strict CommonMark renderers do not auto-link bare URLs, and the line's purpose is to render as a real `<a href>` on aggregators (skills.sh, etc.) for SEO backlinks.

**Why two surfaces, two behaviors:**

- Aggregators consume raw `SKILL.md` from GitHub and render the line as a hyperlink. This is the SEO point.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jeffallan/claude-skills](https://github.com/Jeffallan/claude-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
