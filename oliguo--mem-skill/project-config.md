---
trigger: always_on
description: Self-evolving memory and knowledge accumulation system for AI agents. Acts as a persistent 'second brain' that automatically retrieves past experiences, captures best practices, and proactively records successful solutions to a private knowledge base. Use this skill whenever starting any task, opening a new conversation, or triggering any other skill. Supports memory engines (default: built-in JSON/Markdown index; optional: QMD semantic search). Initialize with: /mem-skill init [--mem-engine=qmd
---


# Mem-Skill: Self-Evolving Knowledge System

## Initialization

When the user runs `/mem-skill init`, execute the following setup:

1. Determine the current workspace root directory.
2. Create the directory structure:
   ```
   <workspace>/
   ├── knowledge-base/
   │   └── _index.json
   ├── experience/
   │   └── _index.json
   └── log.md
   ```
3. Populate `knowledge-base/_index.json` with the starter template (see "Knowledge Base Index Format" below).
4. Populate `experience/_index.json` with the starter template (see "Experience Index Format" below).
5. Create `log.md` with the starter template (see "Log Format" below).
6. Append to `log.md`: `## [YYYY-MM-DD] init | mem-skill initialized (engine: default)`
7. Confirm to the user: "mem-skill initialized. Knowledge base, experience, and log created."

### Engine Selection

When the user runs `/mem-skill init --mem-engine=qmd`, optionally with extra `--qmd-*` flags:

**Supported flags (all optional):**
| Flag | Values | Default |
|------|--------|---------|
| `--qmd-scope=<scope>` | `project`, `global` | _(ask user)_ |
| `--qmd-knowledge=<name>` | any string | _(ask user)_ |
| `--qmd-experience=<name>` | any string | _(ask user)_ |
| `--qmd-mask=<pattern>` | glob pattern | `**/*.md` |

**Examples:**
```
/mem-skill init --mem-engine=qmd
/mem-skill init --mem-engine=qmd --qmd-scope=project
/mem-skill init --mem-engine=qmd --qmd-scope=global --qmd-knowledge=my-kb --qmd-experience=my-exp
/mem-skill init --mem-engine=qmd --qmd-mask="**/*.md,**/*.txt"
```

**Init procedure:**

1. Perform all standard init steps above.
2. Check if QMD is installed: run `which qmd` or `npx @tobilu/qmd status`.
3. If QMD is **not** installed, prompt:
   > "QMD is not installed. Install it now with `npm install -g @tobilu/qmd`? (QMD requires Node.js >= 22)"
4. **Determine collection scope:**
   - If `--qmd-scope` was provided, use that value.
   - **Otherwise, you MUST ask the user** (do NOT guess or auto-choose):
     > "Where should QMD collections be stored?
     > 1. **Project** — scoped to this workspace (recommended for multi-project setups)
     > 2. **Global** — shared across all workspaces"
   - If **project**: default collection name prefix is the sanitized workspace folder name (e.g., folder `my-app` → prefix `my-app`).
   - If **global**: default collection name prefix is `mem`.
5. **Determine collection names:**
   - If `--qmd-knowledge` was provided, use that value.
   - If `--qmd-experience` was provided, use that value.
   - **For any name NOT provided via flags, you MUST ask the user** (do NOT auto-generate):
     > "What name for the knowledge collection? (default: `<prefix>-knowledge`)"
     > "What name for the experience collection? (default: `<prefix>-experience`)"
   - Accept user input or use the defaults if the user confirms.
6. **Determine file mask:**
   - If `--qmd-mask` was provided, use that value.
   - Otherwise use `**/*.md`.
7. After all values are confirmed, create the QMD collections:
   ```bash
   qmd collection add <workspace>/knowledge-base --name <knowledge-name> --mask "<mask>"
   qmd collection add <workspace>/experience --name <experience-name> --mask "<mask>"
   qmd context add qmd://<knowledge-name> "General knowledge base: reusable workflows, preferences, best practices"
   qmd context add qmd://<experience-name> "Skill-specific experience: pitfalls, parameters, solutions"
   qmd embed
   ```
8. Create a `.mem-skill.config.json` at the workspace root:
   ```json
   {
     "engine": "qmd",
     "version": "1.0.0",
     "scope": "<project|global>",
     "mask": "<mask>",
     "collections": {
       "knowledge": "<knowledge-name>",
       "experience": "<experience-name>"
     }
   }
   ```
9. Append to `log.md`: `## [YYYY-MM-DD] init | mem-skill initialized (engine: qmd, scope: <scope>, collections: <knowledge-name>, <experience-name>)`
10. Confirm: "mem-skill initialized with QMD memory engine. Collections created and embeddings generated."

**IMPORTANT:** Never silently create QMD collections without confirming scope and names with the user. If no `--qmd-*` flags were provided, every question above MUST be asked interactively.

For the **default engine** (no `--mem-engine` flag), create `.mem-skill.config.json` with:
```json
{
  "engine": "default",
  "version": "1.0.0"
}
```

For detailed engine-specific behavior, see [references/qmd-engine.md](references/qmd-engine.md) and [references/engines.md](references/engines.md).

## Upgrade Command

When the user runs `/mem-skill upgrade`, migrate an existing mem-skill workspace to the latest version. This is safe to run multiple times — it only adds what's missing.

**Procedure:**

1. **Check prerequisites:**
   - Verify `knowledge-base/_index.json` and `experience/_index.json` exist. If not, tell the user to run `/mem-skill init` instead.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oliguo/mem-skill](https://github.com/oliguo/mem-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
