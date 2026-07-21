---
trigger: always_on
description: Tako's protocol is v0: do not keep any legacy code, backward compatibility shims, or deprecated paths — break things freely until protocol v1.
---

# Instructions for AI agents working on the Tako codebase

Tako's protocol is v0: do not keep any legacy code, backward compatibility shims, or deprecated paths — break things freely until protocol v1.

## Key Principles

1. **Keep SPEC.md in sync with code** - Whenever you modify code, update SPEC.md if needed (avoid insignificant implementation details; focus on user-facing behavior and architecture)

2. **Move finalized behavior to SPEC.md** - Keep SPEC.md as the source of truth for implemented behavior; keep planning out of in-repo TODO files

3. **TDD is mandatory for Rust crates and SDK code** - Write tests before implementation. No backend/SDK feature is complete without passing tests. Website (`website/`) changes are explicitly exempt.

4. **Don't over-engineer** - Simple is better than complex. Avoid premature abstractions, extra configurability, or handling scenarios that can't happen

5. **Trust internal code** - Only validate at system boundaries (user input, external APIs). Don't add defensive checks between trusted internal components

6. **Keep README files in sync for touched components** - When code changes setup, commands, or run/test flow, update the relevant README.md files. README content should stay basic and practical (what it is, how to run), not a specification.

7. **Keep SPEC-derived website docs in sync** - The following docs are agent-maintained outputs derived from `SPEC.md` and must be updated whenever `SPEC.md` behavior changes:
   - `website/src/pages/docs/how-tako-works.md`
   - `website/src/pages/docs/tako-toml.md`
   - `website/src/pages/docs/presets.md`
   - `website/src/pages/docs/troubleshooting.md`
   - `website/src/pages/docs/cli.md`
   - `website/src/pages/docs/deployment.md`
   - `website/src/pages/docs/development.md`

8. **Runtime behavior lives in plugins** - Runtime definitions (install commands, launch args, entrypoint paths) are in `tako-runtime/src/plugins/`. Preset definitions live in `presets/`.
   - `presets/{language}.toml` — family preset definitions (sections per preset)

9. **Never commit with known failures** - Do not commit when tests or pre-commit hooks are known to be failing. Fix the issues first. If fixing is impractical, get explicit user confirmation before committing.

10. **Keep files small and focused** - No single source file should grow beyond ~800 lines. When adding code to a file that's already large, split it first. But don't split by line count alone — split by responsibility. Ask "is this the right boundary?" not just "is this file too big?" Each module should have one clear responsibility. Prefer sibling submodules (e.g. `commands/init/scaffold.rs`) over letting a file accumulate unrelated concerns. Tests belong in their own `tests.rs` submodule, not inline in large files.

11. **Don't test against removed code or features** - Never write assertions that check a removed symbol, field, flag, message, or API is absent (e.g. `assert!(!output.contains("OLD_NAME"))`, `expect(...).not.toContain("deprecated-flag")`). Once the feature is gone from the code, it cannot reappear, and the test becomes dead weight that rots over time. Negative assertions are only valid when they encode a _current_ behavioral distinction (e.g. "in CI mode, ANSI colors are suppressed"; "NODE_ENV appears in ProcessEnv but not in TakoBaseEnv"). If you catch yourself adding `!contains(...)` for something that no longer exists anywhere in the codebase, delete the assertion instead.

12. **Write polished user-facing copy on the first pass** - Treat every CLI prompt, success message, warning, error, docs example, and website UI string as product copy. Before implementing or finalizing user-facing text, read the whole flow as the user will see it and tighten it. Copy should be short, natural, specific, and action-oriented. Avoid redundant nouns, repeated context, implementation terms, and stiff phrases when the UI already supplies the object, environment, target, or state. Prefer familiar verbs users would say themselves (`Set`, `Updated`, `Replace`, `Remove`, `Try again`) over internal or abstract wording (`configured`, `environment`, `override`, `operation`, `proceed`). Good copy should usually answer: what happened, what needs attention, or what action is next, with no extra explanation unless it prevents a real mistake.

13. **Use internal CLI output guidance** - Before writing or modifying `tako` CLI output, read `.agents/internal/cli-output.md`. This internal guide is not an installable public skill.

14. **Keep installable skills user-facing** - Skills under `sdk/javascript/skills/`, `website/public/.well-known/agent-skills/`, `skills/`, and `.agents/skills/` are for users building apps with Tako. Do not put Tako maintainer workflow, release process, CI, tagging, or internal implementation policy there. Put maintainer-only guidance in `AGENTS.md`, repo docs such as `scripts/README.md`, or `.agents/internal/` when it is only for agents working on this repo.

## Project Structure

**Rust Crates:**

- `tako-core/` - Shared protocol types (Command, Response enums)
- `tako-runtime/` - Runtime plugins, download engine, caching

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tako-sh/tako](https://github.com/tako-sh/tako) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
