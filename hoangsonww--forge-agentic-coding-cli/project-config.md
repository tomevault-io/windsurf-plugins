---
trigger: always_on
description: Project-wide guidance for any code-writing agent (Codex, OpenAI agent
---

# AGENTS.md

Project-wide guidance for any code-writing agent (Codex, OpenAI agent
runtime, Cursor, Aider, Cline, Continue, …). Claude Code reads this too
but also reads [CLAUDE.md](CLAUDE.md).

> **Tooling infrastructure.** On top of this file, the repo ships
> agent-specific wiring so every major coding agent has the same rules,
> skills, and permissions:
>
> - `.claude/` — Claude Code settings, path-scoped rules, skills
>   (`/verify`, `/add-tool`, `/add-provider`, `/add-agent`,
>   `/debug-loop`, `/release-check`), subagents (`code-reviewer`,
>   `test-runner`, `docs-auditor`), and safe/deny permissions.
> - `.codex/` — Codex `config.toml`, execpolicy `rules/default.rules`,
>   custom agents (`forge_explorer`, `forge_reviewer`,
>   `forge_test_runner`), guardrail hooks, and Codex-native skills.
> - `.cursor/rules/` — Cursor `*.mdc` rules (always-applied core +
>   path-scoped TypeScript/testing/tools/models/security/UI rules).
> - `.agents/skills/` — portable skills following the open
>   [agentskills.io](https://agentskills.io/specification) spec. Any
>   compliant agent can read these without vendor-specific wiring.
>
> If you update a workflow (e.g. the verify chain), update it in
> `.agents/skills/verify/SKILL.md` first, then mirror the change into
> `.claude/skills/verify/SKILL.md` and `.codex/skills/verify/SKILL.md`
> if they diverge.
>
> **Agentic Coding Flywheel.** Forge follows a plan-heavy, bead-driven
> workflow for any non-trivial change. See [FLYWHEEL.md](FLYWHEEL.md)
> for the full methodology. In brief:
>
> - **Plan space** → `.flywheel/plans/` (iterated markdown plans)
> - **Bead space** → `.beads/beads.jsonl` (self-contained work units)
> - **Code space** → `src/` + `test/` (implementation)
>
> Skills: `/plan`, `/plan-synthesize`, `/plan-to-beads`,
> `/polish-beads` (run 4–6×), `/fresh-eyes`, `/dedupe-beads`,
> `/idea-wizard`, `/deep-review`, `/reality-check`, `/landing`,
> `/de-slopify`. Subagents: `bead-polisher`, `plan-synthesizer`,
> `skill-refiner` (Claude); `forge_bead_polisher`,
> `forge_plan_synthesizer` (Codex). The 8 canonical operators live in
> `.flywheel/operators/`.
>
> **Post-compaction ritual.** When an agent gets confused, send: "Reread
> AGENTS.md, CLAUDE.md, and FLYWHEEL.md so they're still fresh in your
> mind." This is the single most common intervention in the
> methodology.

> Conform to the OpenAI `AGENTS.md` convention: this file is a flat
> Markdown cheat-sheet that answers the questions "where am I, what
> can I run, and what shouldn't I break?".

---

## 1. Project identity

Forge is a **TypeScript CLI runtime** for local-first agentic software
engineering. Node 20+. Ships via npm and a multi-arch Docker image.

- Entry point: `bin/forge.js` → `dist/cli/index.js`
- Orchestrator: `src/core/orchestrator.ts`
- Agentic loop: `src/core/loop.ts`

## 2. Commands you'll run

```bash
npm ci --ignore-scripts
npm run build          # tsc + copy-assets
npm test               # vitest; 249 tests must pass
npm run typecheck
npm run lint
npm run format         # writes
npm run format:check   # reads, for CI
npm run test:coverage
./bin/forge.js doctor
```

Always end a change with:

```bash
npm run format && npm run lint && npm run build && npm test
```

## 3. Layout

| Path | Purpose |
|------|---------|
| `src/cli/` | commander-based CLI, REPL, raw-mode input editor |
| `src/core/` | orchestrator, agentic loop, mode policy, validation gate |
| `src/agents/` | planner, architect, executor, reviewer, debugger, memory |
| `src/models/` | providers (ollama/openai/anthropic/llamacpp/vllm/lmstudio), router, adapter, catalog |
| `src/tools/` | 18 tools |
| `src/permissions/` | risk + interactive permission manager |
| `src/sandbox/` | path-safe fs + command risk classifier |
| `src/persistence/` | tasks/sessions/conversations/events + SQLite |
| `src/memory/` | hot/warm/cold/learning |
| `src/ui/` | HTTP + WS dashboard + static app |
| `src/mcp/` | MCP bridge |
| `test/unit/` | vitest unit tests |
| `docs/` | ARCHITECTURE, INSTALL, SETUP, metrics |
| `.github/workflows/` | ci, release, nightly |
| `docker/` | Dockerfile + compose |

## 4. Rules

### Must

- Keep `npm test` at **100%** pass.
- Respect the state machine in `src/persistence/tasks.ts`.
- Gate every new tool through `requestPermission`.
- Classify model ids through `src/models/local-catalog.ts` (don't
  hand-roll regexes in a new provider).
- Add a unit test for any new logic in `src/core`, `src/agents`, or
  `src/tools`.

### Must not

- Bypass the permission system.
- Introduce network calls in tests — use `vi.mock`.
- Log credentials. Use `src/security/redact.ts`.
- Add dependencies without a clear reason.
- Rename exported APIs without updating every caller + the docs.

### Good defaults

- Prefer `readonly` / immutable data flow.
- Prefer function modules over classes unless state really needs
  encapsulation (see the provider classes for the accepted shape).
- Prefer explicit `Result<T, E>`-style shapes over thrown errors for
  expected failures. Throw only for programmer errors.

## 5. Testing

Vitest. Patterns to copy:

- Stubbing `callModel`: see `test/unit/executor-loop.test.ts`.
- Stubbing providers: see `test/unit/adapter.test.ts`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hoangsonww/Forge-Agentic-Coding-CLI](https://github.com/hoangsonww/Forge-Agentic-Coding-CLI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
