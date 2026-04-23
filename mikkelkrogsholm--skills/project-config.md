---
trigger: always_on
description: This project creates Claude Code skills that wrap CLIs built with [bunli](https://github.com/user/bunli). Each skill is **self-contained** — the CLI is bundled inside the skill folder (`skills/<name>/cli/`) so the skill is portable and can be packaged as a `.skill` file via skill-creator's `package_skill.py`.
---

# Skills Factory

This project creates Claude Code skills that wrap CLIs built with [bunli](https://github.com/user/bunli). Each skill is **self-contained** — the CLI is bundled inside the skill folder (`skills/<name>/cli/`) so the skill is portable and can be packaged as a `.skill` file via skill-creator's `package_skill.py`.

## Development Method: Documentation-Driven + Test-Driven

All work is delegated to sub-agents. The main conversation acts as orchestrator — monitoring progress, tracking phases via the **task system** (`TaskCreate`/`TaskUpdate`), and never doing implementation work directly.

When the user points at an online source (website, API, feed, etc.), the orchestrator creates tasks for each phase, sets up dependencies, and delegates to sub-agents. Phases run sequentially unless explicitly noted as parallelizable.

---

## Phases

### Phase 1: Research

Research is split into two steps. The orchestrator decides after Step 1 whether Step 2 is needed.

**Step 1: Scout (single fast agent)**
- Opens the site, checks for sitemap.xml, robots.txt
- Gets a high-level overview: what sections exist, what the site offers
- Returns a structured list of areas to research
- Writes findings to `research/<name>/scout.md`

**Step 2: Deep research (parallel agents) — only if needed**
- One agent per area identified by the scout
- Each agent writes findings to `research/<name>/<area>.md`
- Each agent reads `.agents/skills/agent-browser/SKILL.md` first and starts its own browser session with a unique name (`--session <name>-<area>`)

**When to skip Step 2:** The source has a clean public REST/JSON API, the scout covered all areas in one pass, and the scout's findings are sufficient to draft the README. This is the common case.

**When to use Step 2:** The source has 4+ distinct feature areas, requires auth investigation, has complex GraphQL schemas, or the scout identified gaps it could not resolve.

**Done when:** `research/<name>/scout.md` exists and the orchestrator has read and confirmed it covers all necessary areas.

---

### Phase 2: Documentation (sub-agent)

Write the "contract" that the CLI must match:

- `skills/<name>/cli/README.md` — full CLI docs with every command, every flag, and **exact JSON output shapes** (test these by running the API manually if possible)
- `skills/<name>/SKILL.md` — the skill wrapper (see Skill Conventions below)
- `skills/<name>/cli/package.json` — dependencies
- `skills/<name>/cli/tsconfig.json` — TypeScript config

The README is the source of truth. Every JSON shape, flag name, and error format documented here must be exactly what the CLI produces.

**Done when:** `bun install` succeeds in the package directory and all documentation files exist.

---

### Phase 3: Tests (sub-agent) — MUST wait for Phase 2

Tests are written based on Phase 2's README — not the research directly. This prevents doc/implementation drift.

- Place tests in `tests/commands/<command>.test.ts`
- Create a shared `tests/helpers.ts` with `runCLI(args)` and `parseJSON<T>(result)` helpers
- Tests run against the live API (no mocking) with small `--per-page` values
- Assert on the exact JSON shapes documented in the README
- Cover: basic output shape, filters, pagination, sort, format flags, error cases, edge cases
- Avoid vacuous tests — ensure assertions actually exercise real data fields

**Done when:** Test files compile and `bun test` exits (tests will fail since CLI isn't built yet, but they should parse correctly).

---

### Phase 4: Build CLI (parallelized per command)

Building is split into two steps to maximize parallelization:

**Step 1: Scaffold (single agent)**
- Create `src/helpers.ts` with `BASE_URL`, `apiFetch`, shared formatters
- Create `src/cli.ts` skeleton that imports and registers all commands (commands can be stubs initially)
- Run `bun install`
- This step is fast and unblocks all parallel command agents

**Step 2: Build commands (parallel agents — one per command)**
- Each agent gets ONE command to build and ONE test file to satisfy
- Each agent reads: the README (for its command's contract), its test file, `helpers.ts`, and the bunli skill
- Each agent writes its command file in `src/commands/<name>.ts`
- Each agent runs only its test: `bun test tests/commands/<name>.test.ts`
- Each agent iterates until its tests pass
- All agents run simultaneously since commands are independent files

**Step 3: Integration check (orchestrator)**
- After all command agents complete, run `bun test` (full suite) to catch any conflicts
- Run `bun run typecheck` (tsc --noEmit)
- Fix any integration issues

**Done when:** Full `bun test` all green + `bun run typecheck` clean.

---

### Phase 5: Validation (sub-agent)

Verify that everything is internally consistent:

- Run each command and compare actual JSON output against README docs
- Check that SKILL.md command signatures match the real CLI
- Check that flag names, types, and defaults are consistent across commands
- Fix any drift found
- Register the skill in `skills-lock.json`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mikkelkrogsholm/skills](https://github.com/mikkelkrogsholm/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
