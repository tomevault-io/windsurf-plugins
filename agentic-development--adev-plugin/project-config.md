---
trigger: always_on
description: <!-- AUTO-SYNCED from .context-index/constitution.md by /adev:sync.
---

# CLAUDE.md

<!-- AUTO-SYNCED from .context-index/constitution.md by /adev:sync.
     Do not edit the synced sections below directly — edit the constitution instead,
     then run /adev:sync to propagate changes. -->

## Identity

adev-plugin is a Claude Code plugin and zero-dependency CLI that implements a full-lifecycle Agentic Development Framework, providing markdown-based skills, bash hooks, and templates for scaffolding structured project context.

## Non-Negotiable Principles

1. **Minimize external dependencies** — prefer Node.js built-ins (`fs`, `path`, `child_process`, `crypto`, `node:test`). Justify any new dependency in an ADR.
2. **Skills are primarily markdown** — skill files are structured instructions for Claude. Companion code (helpers, validators) is allowed but must not be required for the skill to function.
3. **Pure ESM** — all `.mjs` files, `"type": "module"` in package.json. No CommonJS.
4. **Hook protocol compliance** — hooks read JSON from stdin + env vars, exit 0 (allow) or 2 (block), output JSON to stdout.
5. **Version parity** — `package.json` and `.claude-plugin/plugin.json` versions must always match.

## Coding Standards

### Language and Runtime

- **Primary language:** JavaScript (ESM, `.mjs` extension)
- **Runtime:** Node.js
- **Package manager:** npm

### Conventions

- **Naming:** camelCase for functions/variables, kebab-case for files and directories
- **File structure:** skills in `skills/<name>/SKILL.md`, hooks in `hooks/`, templates in `templates/`, CLI in `cli/`
- **Import ordering:** Node.js built-ins first, then relative imports
- **Error handling:** hooks use exit codes (0 = allow, 2 = block); CLI uses `process.exit(1)` for fatal errors
- **Logging:** hooks output JSON to stdout; CLI uses `console.log` for user-facing output

### Commit Trailers

Commits that implement or fix spec-tracked work **must** include a `Spec:` trailer linking back to the spec file. This enables `/adev:retro` and `/adev:hygiene` to trace commits to lifecycle artifacts.

**Required format:**
```
feat(<module>): <description>

Spec: .context-index/specs/features/<module>/<spec-slug>.spec.md
Plan-task: <task-number>          # when implementing a plan task
```

**When to add trailers:**
- `/adev:implement` — always (Spec + Plan-task)
- `/adev:debug` — when fixing a bug tracked by a spec (Spec only)
- `/adev:validate` — when auto-fixing issues (Spec only)
- Manual commits touching spec-tracked code — Spec trailer recommended

**Existing required trailers** (from manifest `provenance` section): `Author-type`, `Operator` — injected by hooks.

### Patterns to Follow

- Hook scripts read `CLAUDE_TOOL_INPUT_*` env vars and JSON from stdin
- Templates are consumed verbatim by `cpSync()` — changes only affect new scaffolds
- Test helpers (`tests/helpers.mjs`) provide `createTempDir()`, `cleanupTempDir()`, `writeFixture()`, `runHook()`
- Issue storage is shared across git worktrees — adapters resolve the main repo root via `resolveStorageRoot()` in `lib/issues/resolve-root.mjs`, overridable with `tasks.db_path` in manifest

### Anti-Patterns to Avoid

- No CommonJS (`require`, `module.exports`)
- No executable logic inside SKILL.md files
- No hardcoded paths to `~/.claude/` — use the plugin root resolution from `cli/index.mjs`
- No `Run inline Node.js:` step directives, `node --input-type=module -e "..."` heredocs, or `node -e "..."` invocations inside `skills/*/SKILL.md`. Skills name a CLI subcommand (`adev <verb> …`) or a helper script; the helper body lives in `lib/cli/` or `scripts/`.
- No SKILL.md contains both an inline-Node block AND an `adev <verb>` invocation within the same H3 section (the per-step boundary; enforces per-skill atomic migration from the cli-driver-surface charter).
- **Fenced JavaScript in SKILL.md must be descriptive-reference only, never executable directive.** A `` ```javascript `` block that names a lib function (e.g., `import { fooFn } from '<ADEV_ROOT>/lib/...';`) is permitted ONLY as documentation of what the named CLI verb wraps — never as an instruction for the agent to execute. The agent reading a SKILL.md MUST call the corresponding `adev <verb> …` whenever the step has runtime semantics. If a fenced JavaScript block contains control-flow logic (branching, iteration, lookup that drives the next step), that logic belongs inside the CLI verb's implementation, not in skill prose. The cli-driver-surface charter's inline-Node sweep covered executable patterns (`node -e`, `Run inline Node:`); this rule clarifies the boundary for descriptive snippets that the sweep did not address.
- **New skills MUST include a Load Skill Extensions block.** Every new `skills/<name>/SKILL.md` MUST contain a block that invokes `adev skill-ext load --skill <name>` (bare slug, matching the parent directory) and the uniform framing prose: *"The following skill extension instructions apply to this invocation (source: installed domain extensions and/or project-level overrides)."* This is required so project-level and extension-pack instructions can append to the skill's execution context. Coverage is enforced by `tests/skills-extension-coverage.test.mjs`. See `.context-index/specs/cross-cutting/universal-skill-extensions.spec.md`.

## Architecture Boundaries


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentic-development/adev-plugin](https://github.com/agentic-development/adev-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
