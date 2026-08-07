---
trigger: always_on
description: This repository publishes independent Agent Skills plugins from `plugins/`.
---

# Repository Instructions

This repository publishes independent Agent Skills plugins from `plugins/`.
Each plugin installs on its own into Coding Agent.
Authoring, editing, and validating `SKILL.md` files is the main work here.
Treat every skill as a production asset, and apply the rules below whenever you create or change one.

## Repository layout

```text
plugins/<plugin>/
  plugin.json                      Claude-compatible plugin manifest
  .codex-plugin/plugin.json        Codex manifest — byte-for-byte identical to plugin.json
  skills/<skill-name>/    
    SKILL.md                       one focused workflow
    references/                    overflow docs (optional, ≤1 level deep)
    scripts/  assets/              optional bundled files
.claude-plugin/marketplace.json    Claude Code marketplace
.cursor-plugin/marketplace.json    Cursor marketplace
.agents/plugins/marketplace.json   Codex marketplace
```

## Non-negotiables

- Keep `plugins/<plugin>/plugin.json` byte-for-byte identical to `plugins/<plugin>/.codex-plugin/plugin.json`.
- Keep marketplace manifests aligned with the plugin directories.
- Marketplace versions come from `GitVersion.yml`; never hand-edit the `version` field in a manifest.
- Run `dotnet run --project build -c Release` after any change to a manifest, marketplace, or skill, and confirm it passes.
- Never copy text from another repository; rewrite in this repo's style.

## Authoring a skill

A skill teaches an agent to perform one task well with the current best API.

### Frontmatter — exactly three keys

```yaml
---
name: <kebab-case, equals the folder name>
description: <the router — see below>
license: MIT
---
```

### Naming

Use kebab-case optimized for keyword overlap with how a developer phrases the task.
Verb-first (`configuring-…`), gerund (`writing-…`), and topic-noun (`revit-ribbon`) forms are all fine.
Keep the technology prefix (`revit-…`, `dotnet-…`); the prefix stakes a namespace.

### Description is the router

The runtime loads a skill by reading only its `description`.
Write one lead sentence of what the skill produces, then `USE FOR:` triggers, then an optional `DO NOT USE FOR:` clause.
Keep it high-level: state outcomes and intents, never method-name lists. APIs drift, a stale list misroutes the agent.

### Granularity

Give each skill one narrow outcome.
Split a broad topic into focused skills, and add one thin end-to-end skill that references them when they form a pipeline.

### Body template (≤500 lines)

```
# Title
<one paragraph: what it produces; for a wrapper library, that it wraps the raw API for readability>
## When to use             bullets
## When not to use         only when a real collision or limitations exist; otherwise omit
## Workflow
### Step 1: …              numbered; the final step verifies the result
## Validation              - [ ] checklist a reviewer can run or observe
## Common Pitfalls         2-column table: pitfall → correct approach
```

Write load-bearing, copy-pasteable fenced code in the target language.
Add a `// BAD` → `// GOOD` contrast only where it teaches.

## House-style rules — self-check every one

1. **Teach the current best way, not API history.** State the good API positively; never document that an old API is `[Obsolete]`, legacy, or deprecated unless steering the agent away from it is a genuine pitfall.
2. **Exclude only real collisions.** A `DO NOT USE FOR` or `When not to use` entry is valid only when a competent agent would otherwise wrongly pick this skill.
    Never exclude a concern that co-applies; a foundational skill runs alongside focused ones.
3. **Do not cap the toolset.** When the real option set is large, give the principle and a few examples, and signal it is open with `…`, never an exhaustive-looking list the agent treats as complete.
4. **Stay internally consistent.** Keep the frontmatter, body, validation, examples, and pitfalls in agreement.
5. **Ground every snippet in real source.** Verify each type, method, property, and argument order against the library or repo, and invent nothing.
6. **Name the dependency package.** A missing member reads as "the package is not referenced" (for example, requires `Nice3point.Revit.Extensions`).
7. **Keep the structure.** Give each skill section headers, numbered steps for any procedure, and at least one fenced code block unless it is a pure rules or checklist skill.

## Plugin self-containment

Each plugin installs alone; a skill must never depend on a skill in another plugin.

- Reference a sibling skill in the same plugin by its bare skill name (`use revit-ribbon`).
- Reference a capability in another plugin by its concrete API or concept, never by skill name (`use RevitContext.BeginDialogSuppressionScope`, not `use revit-context-access`).
- Keep markdown file links inside the skill directory: at most one level deep, with no `..`, absolute, or repo-rooted paths.

## Writing style

- Open with the fact the reader needs.
- Cut anything an agent can infer from the heading, signature, or surrounding text.
- Describe observable behavior and contracts, not the current implementation.
- Keep one idea per sentence, and write one sentence per line.
- Use numbered steps for procedures and checklists for requirements.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nice3point/revit-skills](https://github.com/Nice3point/revit-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
